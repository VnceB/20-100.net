---
title: "Contributing to Open Source Again, With AI"
date: 2026-03-21
tags: ["open-source", "ai", "mcp", "tiki-wiki", "claude-code"]
description: "How I used Claude Code to build an MCP server for Tiki Wiki and submit my first open source contribution in years."
aliases:
  - /articles/contributing-to-open-source-again-with-ai/
---

> **Heads up:** Much of the prose and code here are AI-assisted, but the ideas are my own.

## Back at it

I used to be active in several open source projects. Life happened, priorities shifted, and at some point I just drifted away from it. No dramatic reason.

The new AI coding tools made me want to try again. The hard part of contributing to open source was never the coding itself, it was the ramp-up: understanding a new codebase, learning its conventions, figuring out which layer to hook into. That's exactly what tools like Claude Code are good at, so I picked something small for a project run by a friend to see how it would go.

## The project: Tiki Wiki

[Tiki Wiki CMS Groupware](https://tiki.org) is a PHP wiki/CMS/groupware app that's been around since 2002. Big, old codebase, the kind where you'd normally spend days reading code before writing any.

I wanted to add [Model Context Protocol](https://modelcontextprotocol.io) (MCP) support. MCP is the open standard that lets AI assistants interact with external tools. WordPress has an MCP integration, Notion has one, Confluence has one. Tiki didn't. The idea is that instead of opening a browser, logging in, navigating to a page, clicking edit, making changes and saving, you just tell your AI assistant "update the homepage" and it handles it over MCP. Read, create, search, edit, delete, view history, all through conversation.

## The process

### Planning

I described what I wanted, an MCP server that exposes Tiki's wiki operations over HTTP, and Claude Code explored the codebase by reading the important files (`tikilib.php`, `console.php`, the wiki service controllers). We put together a plan that identified which Tiki service layer methods to wrap (`create_page`, `update_page`, `get_page_info`, `list_pages`, `remove_all_versions`), the bootstrap pattern to replicate from `console.php`, the PHP MCP SDK to use (`mcp/sdk` by PHP Foundation and Symfony), and 7 tools to implement across read and write operations.

### Implementation

I don't have PHP installed locally so we set up a Docker environment with MariaDB and PHP 8.1 Apache to develop against. Claude Code wrote the PHP files: an HTTP entry point (`mcp.php`) using Streamable HTTP transport, a server wrapper that builds the MCP server from tool classes using PHP 8 attributes, the 7 wiki tools each calling TikiLib's service layer directly, a Bearer token auth provider using Tiki's existing `tiki_api_tokens` table, and an output buffering context to prevent TikiLib from corrupting the JSON-RPC stream.

One thing worth explaining is that the tools call Tiki's PHP service layer in-process rather than going through HTTP. `TikiLib::lib('tiki')->create_page(...)` is a direct function call because the MCP server runs inside the same PHP process as Tiki. Permission checks via `Perms_Context` just work, so whatever the authenticated user can do in the Tiki UI, they can do via MCP and nothing more.

Here's how the pieces fit together:

```
MCP Client (Claude Code, etc.)
  │
  │  HTTP POST + Authorization: Bearer <token>
  ▼
Apache / Nginx
  │
  ▼
mcp.php
  ├── ApiTokenAuthProvider ──► tiki_api_tokens (validate + track hits)
  ├── Perms_Context (enforce user permissions)
  └── MCP SDK (StreamableHttpTransport)
        │
        ▼
      WikiTools (7 tools)
        ├── wiki_list_pages
        ├── wiki_get_page
        ├── wiki_get_page_history
        ├── wiki_search
        ├── wiki_create_page
        ├── wiki_update_page
        └── wiki_delete_page
              │
              │  in-process calls
              ▼
          TikiLib / WikiLib / HistLib
              │
              ▼
          MariaDB / MySQL
```

### The bugs

A few things broke along the way. After calling `update_page()`, the next `get_page_info()` returned the old version because Tiki caches page info in memory and the cache wasn't getting invalidated. The fix was passing `skipCache=true` after write operations. This is the kind of thing buried deep in a 10,000-line file that you find by staring at code for hours, or by having an AI trace the execution path in seconds.

The search was also broken because `list_pages()` defaults to `exact_match=true`, so searching for "MCP" wouldn't find a page called "MCP Test Page" until we passed `false` explicitly. And the MCP SDK turned out to expect `[ClassName::class, 'methodName']` as two strings for tool registration rather than an object and a string, so we had to register tool instances in the SDK's PSR-11 container first.

### Security

After the initial implementation I asked Claude Code to do a security audit of everything we wrote. It found 14 issues, some minor and some not minor at all.

The worst one was that the auth layer fell back to a default admin user when no token was provided. So if you didn't send an Authorization header, you got full admin access. We ripped out the fallback entirely so every request needs a valid Bearer token or gets a 401. MCP sessions were stored in Tiki's `temp/` directory which is inside the web root, so we moved them outside the document root with restricted permissions. The token generation used `uniqid()` which is time-based and predictable, so we replaced it with `random_bytes(32)`. Write operations were logging the IP as `0.0.0.0` instead of the actual client IP. A 403 error response included the username from the token when it should have been a generic message. And Apache CGI/FastCGI strips the Authorization header in some configs, which made token auth silently fail without handling the `REDIRECT_HTTP_AUTHORIZATION` fallback.

The auth fallback one really got to me. That was a "make it work first" shortcut that would have given unauthenticated admin access to anyone who could reach the endpoint. I caught it during review, but it's the kind of thing that easily ships if you're moving fast and not thinking about it.

### Testing

We wrote 27 PHPUnit integration tests covering all 7 tools and the auth layer, running against a real database rather than mocks so they exercise the full path from tool method to TikiLib to MariaDB and back. The suite covers the full CRUD lifecycle from create through delete, error handling for things like duplicate pages and missing pages, input validation with pagination limits and sort mode allowlists, and the full range of auth scenarios including valid tokens, invalid tokens, expired tokens, missing tokens, and hit tracking.

### The result

Once everything was working I connected Claude Code to the MCP server and could manage wiki content through conversation. "What's on the main page?" reads the page, "create a new policy document" creates it with wiki syntax. The tools show up in Claude Code's tool list and work like any other MCP integration. We packaged it as a single commit following Tiki's conventions and submitted a draft merge request on GitLab with a `Co-Authored-By: Claude` trailer for transparency.

## What I learned

The AI absolutely cuts through the ramp-up problem. Tiki's codebase is 20+ years old with legacy procedural code sitting next to modern Symfony stuff, and understanding the bootstrap sequence, the service layer, the permission system and the caching behavior is the kind of thing that takes days of exploration normally. Claude Code did it by reading the source and what would have taken me most of a week took minutes.

But you can't just let it run. The AI wrote code that fell back to unauthenticated admin access, stored sessions in the web root, and used a predictable token generator. I caught the auth issue during review and the others came out in a dedicated security audit pass that we did as a separate phase after implementation. Not "let's hope we catch things during dev" but "stop, read every file, look for problems." That's when the real issues surfaced and I think doing it that way is important.

## What's next

This only covers wiki pages. Tiki has trackers, file galleries, calendars, forums, blogs and a lot more, and each could get its own set of MCP tools following the same pattern. The architecture is there, it's just a matter of adding more tool classes.

If you've been thinking about contributing to open source but the ramp-up keeps stopping you, try it with AI tooling. Pick something small, be upfront about the AI assistance, and review everything before you submit because the AI will absolutely ship security holes if you let it.
