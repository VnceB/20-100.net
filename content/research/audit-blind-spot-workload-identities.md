---
title: "The Audit Blind Spot: Why Workload Identities Escaped Review and Why That Era Is Ending"
date: 2026-04-08
draft: false
description: "Most organizations cannot pass a rigorous audit of their non-human identity controls today. PCI DSS 4.0.1, DORA, and NYDFS now explicitly require service account access reviews, yet the average enterprise still has 60% of its AWS IAM access keys older than one year."
tags: ["NHI", "identity", "compliance", "audit", "AI agents", "DORA", "PCI DSS"]
---

**Independent Research Note | April 2026**

---

## Bottom Line

Most organizations cannot pass a rigorous audit of their non-human identity (NHI) controls today. PCI DSS 4.0.1, DORA, and NYDFS now explicitly require service account access reviews, inventory, and credential hygiene, yet the average enterprise still has 60% of its AWS IAM access keys older than one year (Datadog, State of Cloud Security 2024). The tooling market is real but immature. The AI agent identity problem is arriving faster than the standards to govern it. Security and risk management leaders who treat NHI governance as a 2027 initiative will find themselves remediating audit findings and incident response gaps simultaneously.

---

## Key Findings

1. **PCI DSS 4.0.1, effective March 2025, introduced the first explicit requirements for application and system account access reviews, password management, and interactive login prevention.** QSAs report these controls were not explicitly covered in version 3.2.1, catching many organizations unprepared (Intersec Worldwide, Schellman).

2. **DORA's Regulatory Technical Standards are the most prescriptive NHI governance mandate globally,** requiring financial entities to maintain service account inventories with documented owners, purposes, and periodic reviews at the same frequency as human privileged accounts. Early supervisory examinations in the Netherlands, Germany, and Ireland are requesting these artifacts now.

3. **Every quantitative survey on NHI ownership gaps is vendor-sponsored.** The widely cited "51% have no clear NHI ownership" figure comes from a CSA/Oasis Security survey (n=383, August 2025) where the vendor co-designed the questionnaire. We found no independent data to contradict it, but the number should be treated as directional, not definitive.

4. **AI agent identity governance is arriving before the standards to support it.** NIST's CAISI initiative launched in February 2026 but will not produce finalized standards until 2027 at earliest. Meanwhile, analyst projections suggest 40% of enterprise applications will embed task-specific AI agents by end of 2026.

5. **The CyberArk-Venafi integration is a sales success but not yet a product reality.** Venafi appeared in 9 of CyberArk's top 10 deals in Q1 FY2025. But the technology has had three owners in two years (Thoma Bravo, CyberArk, now Palo Alto Networks), and practitioners at Venafi's own Machine Identity Summit in 2024 were still requesting "one dashboard, please" (Kraft Heinz CISO Ricardo Lafosse).

6. **Credential rotation remains the most common source of self-inflicted NHI outages.** Cloudflare suffered a 67-minute global write outage in March 2025 because a rotation script deployed new credentials to dev instead of production. Microsoft paused key rotation after a previous outage, which contributed to the Storm-0558 signing key compromise. Fear of breaking production is the primary reason organizations do not rotate credentials, and it is not irrational.

---

## Forward-Looking Assumptions

1. **By 2028, 60% of enterprises subject to PCI DSS, DORA, or NYDFS will receive audit findings specifically citing inadequate NHI access reviews,** up from fewer than 15% in 2025. The control language is now unambiguous. Auditor interpretation will catch up to the text.

2. **By 2027, fewer than 20% of organizations deploying AI agents will have implemented agent-specific identity governance controls.** Standards lag is the root cause. Most will retrofit governance after incidents or audit pressure, not before.

3. **By 2029, the standalone NHI governance market will consolidate into three categories: cloud-native platform features (Microsoft, AWS, Google), PAM-adjacent suites (Palo Alto/CyberArk, BeyondTrust), and pure-play NHI posture management for multicloud.** Two of the five current pure-play vendors (Astrix, Oasis, Clutch, Entro, Token Security) will be acquired or fail to reach scale.

4. **Through 2028, the majority of NHI governance programs will be owned by platform engineering or DevOps teams, not IAM or GRC.** This is a problem. Platform teams optimize for velocity; they do not naturally build compliance artifacts. Organizations that do not create a federated governance model with IAM policy oversight will fail audits even if their technical controls are adequate.

5. **By 2027, at least one G7 financial regulator will issue enforcement action explicitly naming NHI management failures as a contributing cause.** The pattern is set: the OCC's own service account breach (disclosed April 2025), the SEC's SolarWinds victim penalties (October 2024), and DORA's supervisory cycle all point to enforcement, not just guidance.

---

## Analysis

### 1. The Audit Gap Is No Longer Interpretive

The compliance landscape shifted in 2024 and 2025. Where frameworks previously used vague language about "all accounts" or "system users," the latest revisions name service accounts, application accounts, and machine identities with specificity that leaves little room for creative interpretation.

**PCI DSS 4.0.1** (effective March 31, 2025) introduced Requirement 7.2.5.1, which requires periodic review of all access by application and system accounts and related access privileges, with management attestation that access remains appropriate. The review frequency is determined through a Targeted Risk Analysis, but the requirement is mandatory. Requirement 8.6.1 goes further: interactive login for system accounts must be prevented unless needed for an exceptional circumstance, and that circumstance must be documented and time-limited. Requirement 8.6.2 prohibits hardcoded passwords in scripts, configuration files, or source code. QSA firm Intersec Worldwide noted these controls were not explicitly covered in version 3.2.1, which left enforcement to individual QSA discretion. That discretion is gone.

**CIS Controls v8.1 Safeguard 5.5** requires organizations to establish and maintain an inventory of service accounts with department owner, review date, and purpose, and to perform reviews at a minimum quarterly. The CIS Assessment Specification enforces this with a gating metric: if the last review was more than three months ago, the control automatically fails. This is the most operationally specific NHI requirement in any major framework.

**NIST SP 800-53 Rev 5** addresses NHI through IA-4 (Identifier Management) and IA-5 (Authenticator Management), both of which explicitly list "service" alongside "individual, group, role, and device." Control enhancement IA-5(7) prohibits embedding unencrypted static authenticators in applications or scripts. Control IA-9 (Service Identification and Authentication) requires unique identification and authentication of system services before establishing communications. These are not new to Rev 5, but their presence gives auditors a direct citation.

**ISO 27001:2022** Annex A 5.15 applies access control to "humans and non-human entities on a network." Annex A 8.2 restricts privileged access for "users, software components, and services." Lead auditors are now interpreting this to include integration accounts, automation scripts, and backup agents (Stuart Barker, ISO 27001 Lead Auditor, HighTable.io).

**SOC 2** remains principles-based. The AICPA Trust Services Criteria do not include a named service account control. But if an organization's system description states it manages "all accounts," auditors will test that claim against service accounts. The trend among Type II auditors is to expand access review testing to include NHIs, particularly under CC6.1 through CC6.3. Organizations whose policies say "all accounts" but whose reviews cover only human users are creating their own audit findings.

We believe the Big 4 audit firms are approximately 12 to 18 months behind the control text in their examination rigor. KPMG published "The Rise of Machine Identities" in 2025, the most visible public statement from any Big 4 firm. A 30-year financial industry veteran writing for the NHI Management Group reported that Big 4 auditor focus on NHI has increased substantially over the last 2-3 years and warned that organizations without NHI programs could face scrutiny soon. But we have not seen standardized Big 4 audit procedures for NHI controls. That gap will close.

### 2. AI Agents Are a New Identity Category, and Nobody Is Ready

AI agents are not service accounts with better marketing. They are fundamentally different: non-deterministic, capable of requesting permission escalation at runtime, able to spawn sub-agents, and designed to operate across multiple systems simultaneously. A service account calls a fixed API with fixed permissions. An AI agent decides at runtime which APIs to call based on reasoning. This distinction matters for identity governance because every assumption about static privilege assignment breaks down.

SailPoint's 2025 survey found that 80% of organizations using AI agents observed them acting unexpectedly or performing unauthorized actions. That number should alarm anyone building agent-based workflows without identity controls.

The hyperscalers are responding, but unevenly. **Microsoft Entra Agent ID** is the most ambitious effort: a new identity primitive for agents with blueprints, an agent registry, conditional access policies, lifecycle governance with human sponsors, and identity protection. It is also still in public preview as of April 2026 and requires a Microsoft 365 Copilot license with Frontier enabled. The vision is right. The production readiness is not there yet. **AWS Bedrock AgentCore Identity** reached general availability in October 2025 with a more pragmatic, infrastructure-centric approach: existing IAM roles extended with agent-specific attributes, OAuth credential providers for third-party service access, and a Cedar-based policy engine in preview. AWS has no lifecycle governance or sponsor concepts. **Google Cloud's Agent Identity** (preview) takes the strongest credential security approach with mTLS-bound certificate tokens that prevent replay attacks, but has the least mature governance layer. None of the three platforms fully addresses sub-agent spawning, delegation chain accountability, or cross-platform identity federation.

The OWASP NHI Top 10, published in 2025, was the first structured risk taxonomy for non-human identities. It ranks Improper Offboarding as the top risk, followed by Secret Leakage and Vulnerable Third-Party NHI. However, it focuses almost entirely on traditional NHIs. The OWASP Top 10 for Agentic Applications, released in December 2025, addresses agent-specific risks but does not connect them to identity governance frameworks. The two lists exist in parallel without integration.

NIST's CAISI initiative, launched February 2026, is the first U.S. government program dedicated to AI agent standards. It includes an NCCoE concept paper on AI agent identity and authorization that identifies prompt injection and accountability gaps as leading vulnerabilities. But the Request for Information comment period closed in March 2026, listening sessions are scheduled for April 2026, and finalized standards are expected in 2027 at the earliest. The majority of first-generation enterprise agent deployments will go live before any NIST agent-specific standard exists. We do not yet know what good agent identity governance looks like in practice, and honesty about that gap is more useful than a premature framework.

### 3. The Vendor Market Is Funded, Fragmented, and Early

Workload Identity Management appeared as a distinct category in the 2025 Gartner Hype Cycle for Digital Identity, the first year it was recognized. Based on available evidence, the category sits near the Innovation Trigger or early Peak of Inflated Expectations. KuppingerCole published its first Leadership Compass for Non-Human Identity Management in 2025, formally establishing NHIM as a market segment. The analyst recognition is real. The market maturity is not.

**Funding is substantial but frequently overstated.** The "$400M+ in H1 2025" figure circulating in vendor marketing originates from a Doppler blog post (Security Boulevard, August 2025) and refers to all of 2025, not just H1. It also includes adjacent vendors beyond the NHI pure-plays. Our verified tally of pure-play NHI funding rounds through 2025: Astrix Security Series B ($45M, December 2024, Menlo Ventures), Oasis Security Series A plus extension ($75M total, 2024, Sequoia/Accel), Clutch Security Series A ($20M, 2024, SignalFire), Entro Security Series A ($18M, 2024, Dell Technologies Capital), Token Security Seed plus Series A ($27M total, through January 2025, TLV Partners/Notable Capital), and Defakto (formerly SPIRL) Series B ($30.75M, October 2025, XYZ Venture Capital). That totals approximately **$216M in verified pure-play rounds through 2025.** Adding Oasis's reported $120M Series B in March 2026 and NHI-adjacent vendors (Aembit, Veza, Natoma) pushes past $400M across 2024 through early 2026. The investment thesis is clear. The "$400M in H1 2025" framing is marketing.

**Practitioner feedback is thin.** Astrix has the most independent validation: named a Cool Vendor by Gartner (2023), RSA Innovation Sandbox finalist, named sample vendor in the 2025 Hype Cycle, and 15.3% PeerSpot mindshare in NHIM as of March 2026. Gartner Peer Insights reviewers praise detection of leaked tokens and service account discovery but note onboarding could be smoother. Entro Security has the richest Gartner Peer Insights reviews, with users calling it an integral part of their cybersecurity strategy alongside a request for deeper AI-driven analysis capabilities. Oasis Security's PeerSpot mindshare declined from 16.8% to 12.8% year over year, a signal worth watching for a vendor that exited stealth in January 2024. Clutch Security, Token Security, and Natoma have essentially no independent public reviews. **CISOs evaluating these vendors are making bets on roadmaps, not proven production outcomes.** Discovery and posture management are the proven use cases. Lifecycle management and threat detection are emerging. AI agent governance is aspirational across the board.

The CyberArk-Venafi story deserves particular scrutiny. CyberArk acquired Venafi for $1.54 billion in late 2024. Sales integration was immediate: Venafi appeared in 9 of CyberArk's top 10 deals in Q1 FY2025. Product integration was targeted for 2025 (CLM plus secrets management) and 2026 (unified human and machine identity platform). Then Palo Alto Networks acquired CyberArk for $25 billion, closing in February 2026. Venafi has now had three owners in two years. Documentation still lives at docs.venafi.com. A Doppler competitive analysis noted that CyberArk's architecture leans toward static identity models, which can be a poor fit for modern environments where machine identities are constantly created and destroyed. We believe the unified machine identity platform vision is now on Palo Alto Networks' roadmap, not CyberArk's, and should be evaluated on a 2028 or later timeline.

### 4. Credential Rotation Works in Theory and Breaks in Production

The operational reality of NHI governance is less about tool selection and more about a specific, uncomfortable question: do you know what will break if you rotate this credential? Most organizations cannot answer it.

Datadog's State of Cloud Security 2024, based on telemetry from thousands of organizations, found that 46% of organizations still use unmanaged users with long-lived credentials. Among Google Cloud service accounts, 62% have access keys older than one year. Among AWS IAM users, 60% have keys older than one year. Andrew Krug, Datadog's Head of Security Advocacy, concluded that it is unrealistic to expect long-lived credentials can be securely managed. This is the most authoritative data point available because it is based on actual infrastructure telemetry, not survey self-reporting.

The fear of rotation-induced outages is well-founded. Cloudflare's March 2025 R2 outage lasted 67 minutes and caused 100% write failures globally because a rotation script omitted the `--env production` flag, deploying new credentials to dev while old credentials were deleted from production. Microsoft paused its key rotation process after a previous outage, which contributed to the delay that enabled the Storm-0558 signing key compromise. The Oasis Security team documented a Dropbox incident in 2024 where emergency rotation of an unrotated AD service account caused partial end-user disruption. CyberArk's own 2025 survey (n=1,200, vendor-sourced) found 72% of organizations experienced certificate-related outages in the past 12 months, up from 45% reporting weekly outages compared to 12% in 2022.

The practitioner consensus, visible across Reddit, security conference talks, and vendor-neutral blogs, follows a consistent failure pattern: an identity is created for an immediate need, permissions are broadened to avoid friction, credentials persist because rotation feels risky, access reviews get rubber-stamped to avoid outages, and nobody is confident enough to remove anything. The CSA blog (February 2026) articulated the structural problem: access reviews were designed to answer a human-centric question about whether a person still needs access. That question does not translate to NHIs without dependency mapping, usage telemetry, and owner accountability.

**SPIFFE/SPIRE** represents the most credible path to eliminating long-lived secrets for workload authentication. Both are CNCF Graduated projects with named production adopters including Uber, GitHub, Square (Block), and Pinterest. HPE invested heavily in core development through its Scytale acquisition. But Defakto Security states that small-scale deployments take 6 to 12 months and complex deployments require 12 to 24 months with a core team of experts. Legacy systems that cannot speak mTLS remain a hard blocker. Cloud-native workload identity federation (AWS IAM Roles, Azure Workload Identity Federation, GCP Workload Identity) works within a single cloud but fragments across multicloud and hybrid environments. The secure path is clear. The migration path for legacy environments is not.

### 5. Nobody Owns NHI Governance, and the Surveys Proving It Are All Vendor-Funded

The CSA/Oasis Security survey (January 2026, n=383) found that 51% of organizations reported no clear ownership or accountability for NHI governance. Oasis financed the project and co-designed the questionnaire. A separate CSA/Astrix survey (September 2024, n=818) found that only 15% feel highly confident in preventing NHI attacks and only 20% have formal NHI offboarding processes. Astrix sponsored that survey. CyberArk's 2025 Identity Security Landscape report (n=2,600) found that 88% define "privileged user" as solely human identities. CyberArk funded it. Entro's 2025 analysis found 97% of NHIs have excessive privileges. Entro is a vendor. The Keeper Security RSAC 2026 survey (n=109) found 76% of NHIs are not governed under privileged access policies. Keeper is a PAM vendor with a convenience sample of 109 conference attendees.

**We found no fully independent, non-vendor-sponsored survey that specifically quantifies NHI governance ownership gaps.** The closest is the 2021 Ponemon/Keyfactor State of Machine Identity Management report (n=1,162), which used Ponemon's independent methodology but was funded by Keyfactor and focused on certificates and keys rather than NHI governance ownership. The directional finding across all sources is consistent: NHI ownership is fragmented and unclear in most organizations. But the specific percentages should be treated as approximate. This is a data gap the analyst community has not filled.

In practice, HashiCorp (now IBM) observed that platform teams are often the de facto NHI managers: the people handling cloud IAM roles for microservices and certificate rotation for service mesh are typically platform engineers, not IAM staff. GitGuardian's analysis found that the person who can answer most questions about an NHI is the developer who created it, which does not mean they are responsible for rotation or lifecycle management. The Gartner IAM Summit in December 2025 advocated a layered identity fabric model where Access Management, IGA, and PAM are interdependent layers that must work together rather than compete for ownership or visibility. ServiceNow added Non-Human and Agentic Identity Governance as a new domain in its IAM capability map, recommending organizations start with NHI discovery and ownership attribution.

Legacy PAM tools have structural limitations for NHI use cases. They are vault-centric (assuming persistent accounts), human-speed (manual checkout workflows), session-oriented (recording RDP/SSH, not API calls), and centralized (requiring vault connectivity). Ephemeral cloud workloads, serverless functions, and CI/CD pipelines do not fit this architecture. The CSA/Astrix 2024 survey found that 54% of respondents use PAM for NHI, but these tools are not specifically designed to address NHI security challenges. BeyondTrust and Delinea face similar architectural gaps. The Palo Alto Networks acquisition of CyberArk signals that PAM is becoming an infrastructure-level control rather than a standalone tool category, but the product integration timeline extends well past 2027.

### 6. Regulators Are Building the Enforcement Case

No U.S. or EU regulation uses the term "non-human identity." But the enforcement infrastructure is forming around the underlying concepts, and the gap between regulatory intent and explicit NHI language is closing fast.

**DORA** is the most important development for financial sector CISOs. The Regulatory Technical Standards on ICT Risk Management, effective January 2025, require financial entities to maintain an inventory of service accounts, document the purpose of each, assign a human owner, prevent interactive use where not required, and review access at the same frequency as human privileged accounts. MFA for privileged accounts is legally mandated, not optional. Early supervisory examinations in the Netherlands, Germany, and Ireland are requesting these artifacts. The service account inventory requirement is, per practitioner reports, one of the most commonly missed items in initial DORA readiness assessments.

**The FFIEC's 2021 guidance** on Authentication and Access remains the most NHI-explicit U.S. regulatory document. Footnote 2 defines "users" to include employees, third parties, service accounts, applications, and devices. The guidance requires authentication considerations for system-to-system communications. The irony is painful: the OCC, which issued this guidance jointly with other banking regulators, disclosed in April 2025 that its own Microsoft 365 tenant was compromised through a service account with administrative privileges that lacked MFA. The breach persisted for 20 months. It was reported to Congress as a major information security incident.

The SEC has not cited NHI by name, but its October 2024 enforcement actions against four SolarWinds victim companies penalized the minimization of credential compromise in public disclosures. Unisys paid $4 million for describing risks as hypothetical despite knowing that seven network credentials and 34 cloud-based accounts were compromised. Mimecast paid $990,000 for failing to disclose that a threat actor exfiltrated an authentication certificate used by approximately 10% of its customers. The FTC's Drizly consent order (January 2023) explicitly cited failure to securely store AWS and database login credentials and failure to scan repositories for unsecured credentials such as usernames, passwords, API keys, secure access tokens, and asymmetric private keys. Personal liability was imposed on the CEO.

**NYDFS 23 NYCRR 500** merits specific attention. The amended regulation, fully effective November 2025, requires Class A companies to implement PAM solutions and mandates MFA for any individual accessing any information system. There is a carve-out for service accounts that prohibit interactive login, but this creates risk rather than eliminating it. Organizations must ensure non-interactive service accounts truly cannot be used interactively, and must implement compensating controls.

The **EU AI Act** (effective in stages through 2027) requires that AI systems interacting with natural persons disclose they are AI (Article 50, effective August 2026) and that high-risk AI systems be registered in an EU database. But the Act is silent on how AI agents authenticate to other systems, manage credentials, or handle privilege escalation. It addresses "is this an AI?" but not "what can this AI access?" That gap will force supplementary regulation as autonomous agents proliferate.

---

## Recommendations for Security and Risk Management Leaders

1. **Inventory first, tool second.** Before evaluating NHI vendors, run a discovery sprint using cloud-native tools (AWS IAM Access Analyzer, Azure Entra Workload ID, GCP Policy Analyzer) and open-source scanners (GitGuardian, TruffleHog) to establish baseline counts of service accounts, API keys, OAuth tokens, and certificates. You cannot govern what you have not counted. Set a 90-day deadline for initial inventory. Do not let it become a permanent project.

2. **Map NHI controls to the audit frameworks you face now.** PCI DSS 4.0.1 Requirements 7.2.5.1 and 8.6.1 through 8.6.3 are enforceable today. CIS Safeguard 5.5 requires quarterly reviews. DORA requires service account inventories with owners. Build the compliance artifact first, then automate it. If your organization is subject to DORA, treat the service account inventory as a regulatory deliverable with a named executive owner and a board-reportable status.

3. **Adopt a federated ownership model, not a centralized one.** IAM or GRC sets policy and defines standards. Platform engineering implements controls in CI/CD pipelines and manages cloud-native NHIs. Application teams own their application-specific NHIs with documented accountability. A central governance function (within the CISO organization) provides visibility, risk scoring, and audit coordination. No single team can own NHI governance at scale. The question is not who owns it but whether the accountability chain is documented and enforceable.

4. **Separate the AI agent identity problem from the NHI backlog.** Agent identity governance requires different controls: human sponsors, runtime behavioral monitoring, delegation chain tracking, and scope enforcement. Do not wait for the NHI backlog to be clean before addressing agent identity. Stand up an agent identity working group now that includes IAM, data science, platform engineering, and legal. Align to CSA's AI Controls Matrix (July 2025) and NIST IR 8596 (draft) as provisional frameworks until NIST CAISI publishes finalized standards.

5. **Evaluate pure-play NHI vendors for discovery and posture management only.** That is where production-validated value exists today. Lifecycle management, threat detection, and agent governance are emerging capabilities without meaningful practitioner validation. Run a 90-day proof of value focused on discovery accuracy, ownership attribution, and secrets exposure detection. Do not sign multiyear platform commitments for capabilities that are still roadmap.

6. **Begin SPIFFE/SPIRE evaluation for net-new workloads; do not attempt a legacy migration simultaneously.** Workload identity federation eliminates long-lived secrets within a single cloud, but legacy systems that cannot speak mTLS remain a hard blocker. Apply secretless architectures to new Kubernetes-native and cloud-native workloads first. Use vaulted credentials with automated rotation for legacy systems. Accept that the transition will take 24 months or more and plan accordingly.

---

## Market Outlook

The NHI governance market sits at a crossroads familiar to anyone who watched CASB, CSPM, or CIEM develop. There is genuine enterprise demand, validated by real audit pressure and breach history. There is real funding, with over $200 million in verified pure-play rounds through 2025. And there is significant risk of premature consolidation and vendor overreach.

The cloud hyperscalers are building native workload identity capabilities that will commoditize intra-cloud NHI authentication over the next three years. Microsoft's Entra Agent ID, if it reaches general availability, could redefine expectations for agent identity governance within the Microsoft ecosystem. AWS AgentCore Identity is already GA. But none of the hyperscalers will solve cross-cloud NHI governance, secrets posture management, or multicloud lifecycle orchestration. That is the defensible wedge for pure-play vendors, and the ones that focus there will survive consolidation.

PAM vendors are rebranding around identity security. The Palo Alto Networks acquisition of CyberArk for $25 billion, combined with the prior CyberArk acquisition of Venafi for $1.54 billion, creates the largest identity security portfolio in the market. BeyondTrust exceeded $400 million ARR in June 2025 and acquired Entitle for CIEM. Delinea acquired Authomize and Fastpath. These are platform plays, not feature additions. But platform integration takes years, and CISOs should not confuse acquisition announcements with product delivery.

The honest assessment is this: we are in the first inning of NHI governance as a discipline. The compliance drivers are real and accelerating. The breach history validates the risk. The tooling is early but improving. The AI agent dimension adds urgency and complexity that the market has not yet absorbed. Organizations that start now, with inventory, ownership, and framework alignment, will be positioned to adopt better tooling as it matures. Organizations that wait for the market to settle will find themselves explaining to auditors, regulators, and boards why they treated machine identities as an afterthought while those identities outnumbered their employees 82 to 1.

---

## Key Sources

**Compliance frameworks:** PCI DSS 4.0.1 (PCI SSC, 2024); CIS Controls v8.1 Assessment Specification (CIS, 2024); NIST SP 800-53 Rev 5 (NIST, 2020); ISO 27001:2022 (ISO/IEC, 2022); DORA RTS on ICT Risk Management (ESAs, 2024); NYDFS 23 NYCRR 500 (NYDFS, amended 2023); FFIEC Authentication and Access Guidance (2021).

**Data sources:** Datadog State of Cloud Security 2024; CSA/Oasis Security State of NHI Security Survey (January 2026, n=383); CSA/Astrix State of NHI Security Report (September 2024, n=818); CyberArk Identity Security Landscape 2025 (n=2,600); Keeper Security RSAC 2026 Survey (n=109); Ponemon/Keyfactor State of Machine Identity Management 2021 (n=1,162); SailPoint AI Agent Survey 2025.

**Vendor and analyst sources:** Gartner 2025 Hype Cycle for Digital Identity; KuppingerCole Leadership Compass: Non-Human Identity Management 2025; Gartner IAM Summit December 2025 (Sayers coverage); Astrix Security, Oasis Security, Entro Security, Clutch Security, Token Security, Defakto funding announcements; CyberArk Q1 FY2025 earnings; Palo Alto Networks/CyberArk acquisition filings; Doppler NHI Platform Comparison (August 2025).

**Incident and enforcement sources:** Cloudflare R2 Outage Post-Incident Report (March 2025); OCC Major Information Security Incident Disclosure (April 2025); SEC v. Unisys, Avaya, Check Point, Mimecast enforcement actions (October 2024); FTC v. Drizly consent order (January 2023); Harvard Law School Forum on Corporate Governance analysis of SEC cybersecurity enforcement (October 2024).

**Standards and frameworks:** OWASP NHI Top 10 (2025); OWASP Top 10 for Agentic Applications (December 2025); NIST CAISI Initiative RFI (February 2026); CSA AI Controls Matrix (July 2025); NIST IR 8596 draft; ServiceNow IAM Capability Map 2025.
