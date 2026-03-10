---
title: "Scaling OpenClaw's Agentic AI to Multi-User Environments with OpenFGA Authorization"
date: 2026-03-10
tags: ["openclaw", "openfga", "authorization", "ai-agents", "security"]
---

> **Heads up:** Much of the prose and code here are AI-assisted, but the ideas are my own.

## The problem

Last week, CodeWall's security agent [broke into McKinsey's AI platform Lilli](https://codewall.ai/blog/how-we-hacked-mckinseys-ai-platform) in two hours. No credentials. A SQL injection on an unauthenticated endpoint gave them read/write access to 46.5 million chat messages, 57,000 employee accounts, and the system prompts controlling the AI's behavior. One SQL UPDATE could have silently rewritten those prompts with no audit trail. The authorization layer around the AI was nonexistent.

That got me thinking about OpenClaw, which I use to run my own AI agents. Its security model is simple: one operator per gateway. If you're the only person using it, that's all you need.

But if three people want to share a gateway, you can't give them different permission levels. Alice should have full control, Bob should be able to use agents but not change settings, Charlie should only be able to watch. Right now? You'd run three gateways. That's not really a solution.

Same problem with community skills. If someone publishes a skill on ClawHub, you either trust it completely or you don't install it. There's no middle ground where an operator reviews it and unlocks it for specific users. And if you want agent A to call agent B's tools, there's no way to say "yes, but only these tools, and only if someone approved it."

The missing piece is that these are all relationship questions. "Can this user invoke this tool via this agent?" depends on who, what, and whether there's a delegation chain. You can't answer that with a boolean config flag.

## Why OpenFGA

I've been wanting to try OpenFGA on a real project for a while, and OpenClaw's multi-user problem seemed like a good fit. It's based on Google's Zanzibar (the system behind Google Drive sharing, YouTube permissions, etc.), it's CNCF-incubating, and it does exactly one thing well: relationship-based access checks.

OpenFGA stores relationships as tuples. "Alice is an operator of the main gateway" is a tuple. "The orchestrator agent can delegate to the researcher agent" is a tuple. You write tuples, then ask questions like "can Alice invoke the deploy tool?" and OpenFGA walks the relationship graph to figure it out. The transitive logic that turns into spaghetti if-else chains in application code becomes declarative.

## The authorization model

This is the full model in OpenFGA's DSL. It gets written to the OpenFGA store when the gateway boots:

```
model
  schema 1.1

type user
  relations
    define exists: [user]

type team
  relations
    define member: [user]

type gateway
  relations
    define operator: [user]
    define admin: [user]
    define user: [user]
    define viewer: [user]
    define can_admin: operator or admin
    define can_use: can_admin or user
    define can_view: can_use or viewer

type agent
  relations
    define exists: [agent]
    define parent_gateway: [gateway]
    define operator: operator from parent_gateway
    define allowed_user: [user, team#member]
    define can_delegate: [agent]
    define can_use: operator or allowed_user

type tool
  relations
    define parent_agent: [agent]
    define allowed_user: [user, team#member]
    define denied_user: [user, team#member]
    define can_invoke: can_use from parent_agent but not denied_user

type skill
  relations
    define publisher: [user]
    define allowed_gateway: [gateway]
    define trusted: [user]
    define can_use: [user]
    define can_install: allowed_gateway

type session
  relations
    define owner: [user]
    define participant: [user]
    define parent_agent: [agent]
    define can_read: owner or participant
    define can_write: owner or participant

type agent_delegation
  relations
    define source_agent: [agent]
    define target_agent: [agent]
    define approver: [user]
    define allowed_tool: [tool]
    define can_delegate: approver
```

A few things worth calling out:

Gateway privileges form a chain: `operator > admin > user > viewer`. Each level includes the permissions of the levels below it through computed relations. A viewer can read. A user can read and invoke tools. An operator can do everything.

Tool access uses a `difference` userset. `can_invoke` grants access to anyone who can use the parent agent, minus anyone on the deny list. So you can block Bob from the `rm -rf` tool without revoking his access to the agent itself.

Tools inherit from their parent agent via `tupleToUserset`. If you can use the agent, you can use its tools, unless someone explicitly denied you.

## What it looks like in practice

I built three things on top of this model, with Claude Code doing most of the heavy lifting on implementation. Here's each one with the actual tuples and what users see.

### Multi-user teams

Say Alice is the operator, Bob is a regular user, Charlie can only watch, and Dave is some random who wandered into the channel. They're all on Slack.

On boot, the gateway reads `openclaw.json` and writes these tuples:

```
user:alice   | operator | gateway:main
user:bob     | user     | gateway:main
user:charlie | viewer   | gateway:main
```

Alice asks the agent to change how deep research runs:
```
[Alice -> Slack] set the deep research thinking budget to high
[MyAgent -> Slack] Done. Deep research will now use extended thinking.
```
She's an operator. The `can_use` check resolves through `operator -> can_admin -> can_use`. She can invoke tools and change agent configuration.

Bob asks the agent to run a deep research task:
```
[Bob -> Slack] deep research: there are thousands of passwordless
MFA vendors out there, why are new ones still popping up every
month, and why are we all still using passwords?
[MyAgent -> Slack] Starting deep research... Searched 31 sources.
Summary: adoption is blocked by legacy system integration costs,
not technology. New vendors keep launching because enterprise
procurement cycles create recurring market windows. Full report
attached.
```
He's a `user`, which feeds directly into `can_use`. He can invoke tools normally, but admin operations (like changing agent configuration) would be denied.

Charlie tries to launch his own research:
```
[Charlie -> Slack] deep research on passkey adoption rates
[MyAgent -> Slack] Authorization denied: you do not have permission
to use the "deep_research" tool on this agent. Contact an agent
admin to request access.
```

But Charlie can still read Bob's research output:
```
[Charlie -> Slack] what did Bob's MFA research find?
[MyAgent -> Slack] Bob's research found that passwordless adoption
is mostly blocked by legacy integration costs, not the technology
itself. New vendors keep appearing because enterprise procurement
cycles create recurring windows. Full report is in the thread above.
```
Charlie is a `viewer`. He has `can_view` but not `can_use`, so the agent won't launch tools for him. But he can read results and ask follow-up questions based on what's already in context.

Dave, who has no tuples at all:
```
[Dave -> Slack] hey can you help me
[MyAgent -> Slack] You are not authorized to use this gateway.
```
The `unknownSenderPolicy` config controls whether unknown senders get blocked or allowed with minimal permissions.

Identity resolution maps platform accounts to canonical users:

```json
{
  "session": {
    "identityLinks": {
      "alice": ["slack:U04ABC123"],
      "bob": ["slack:U04DEF456"],
      "charlie": ["slack:U04GHI789"]
    }
  }
}
```

All three are on Slack, each mapped to their canonical user ID. The same identity resolution works across channels if someone is on multiple platforms.

### Skill trust tiers

Skills get a trust level based on their source:

| Source | Trust level | Default |
|---|---|---|
| `openclaw-extra`, `openclaw-bundled` | untrusted | Blocked until explicitly granted |
| `openclaw-managed`, `agents-skills-personal`, `agents-skills-project` | verified | Allowed |
| `openclaw-workspace` | privileged | Blocked until operator approves |

Alice tries to run `curl-fetch`, a community skill from `openclaw-extra`:
```
[Alice -> Discord] /skill curl-fetch https://example.com
[MyAgent -> Discord] The "curl-fetch" skill is from an untrusted
source. Ask an agent admin to grant access.
```

An operator reviews the skill and decides it's fine:
```
[Operator -> Discord] /auth grant user:alice can_use skill:untrusted/curl-fetch
[MyAgent -> Discord] Granted can_use on skill:untrusted/curl-fetch for user:alice
```

That writes one tuple:
```
user:alice | can_use | skill:untrusted/curl-fetch
```

Now it works:
```
[Alice -> Discord] /skill curl-fetch https://example.com
[MyAgent -> Discord] Fetching https://example.com... (200 OK, 12.4KB)
```

Privileged skills (workspace tier) fail closed. If OpenFGA is down, privileged skills get denied rather than allowed. Better to block a privileged operation by accident than to allow one.

### Cross-agent delegation

Say you have an `orchestrator` agent that coordinates work and a `researcher` agent that searches the web. The orchestrator wants to send tasks to the researcher.

Without approval:
```
[orchestrator attempting sessions_send to researcher]
Agent "orchestrator" is not approved to delegate to agent "researcher".
Request approval from a gateway operator.
```

Grant it:
```
[Operator -> Discord] /auth grant agent:orchestrator can_delegate agent:researcher
[MyAgent -> Discord] Granted delegation: orchestrator -> researcher
```

Tuple:
```
agent:orchestrator | can_delegate | agent:researcher
```

Now the orchestrator can send tasks:
```
[orchestrator -> researcher session] Research the latest OpenFGA release
[researcher] Searching... Found: OpenFGA v1.11.6 released Feb 2026...
```

This is one-directional. The researcher can't delegate back to the orchestrator unless that's explicitly granted too. Neither can delegate to a `coder` agent without a separate tuple.

## Conclusion

This started as a "let me see if OpenFGA even works for this" experiment. It went better than I expected. The authorization model maps cleanly to the problems agentic AI platforms actually have: who can use what, which agents can talk to each other, and which community-contributed skills are safe to run. The relationship-based approach fits naturally. I didn't have to fight the model to express what I needed.

The McKinsey/Lilli breach is a reminder that AI platforms are becoming high-value targets, and the authorization layer is often the weakest part. As agents get more capable and start operating in shared environments, the question of "who is allowed to do what" stops being optional. Relationship-based authorization systems like OpenFGA give you a way to answer that question without building a mess of ad-hoc checks.

This is a proof of concept, not production code. There's a detailed breakdown of what's enforced at the code level, what's modeled but not yet wired in, and what's left to build in the [project README](https://github.com/VnceB/openclaw/blob/feature/openfga-authorization/src/auth/README.md). 126 unit tests and 16 e2e tests against a live OpenFGA instance.

## What's next: the Minecraft test

My next step is to put this in front of real users. I run OpenClaw at home where my kids use it to manage our family Minecraft servers. Right now they all have the same access, which means my youngest discovered he can kick his older brothers off the server whenever he feels like it. He thinks it's hilarious. His brothers do not.

This is exactly the kind of problem the authorization model should solve. The oldest should have full server admin access: install plugins, change game rules, manage players. The middle one knows enough to handle day-to-day stuff like installing trusted plugins and tweaking game rules, but shouldn't be kicking anyone. The youngest should only be able to do low-risk things like changing the weather or time of day. And any new tools added later should be blocked for him by default, so there are no more surprise kicks.

That's probably the real test of whether this architecture holds up: not a synthetic Alice/Bob/Charlie scenario, but an actual household where one kid will absolutely find every edge case in your permissions model and exploit it for laughs.

If you're building multi-user agents or thinking about authorization for AI platforms, I'd be interested to hear how you're approaching it.
