---
title: "The End of NTLM and the Kerberos Reckoning"
date: 2026-04-14
draft: false
description: "Microsoft is dismantling NTLM through a three-phase deprecation plan while Kerberos faces its own expanding attack surface. A comprehensive look at the threat landscape, vendor responses, and the modern alternatives enterprises must adopt."
tags: ["identity", "security", "NTLM", "Kerberos", "Active Directory"]
---

**Independent Research Note | April 2026**

---

## Bottom Line

Microsoft is dismantling NTLM, the 30-year-old authentication protocol still used by 64% of Active Directory accounts, through a three-phase deprecation plan that will disable it by default in the next major Windows Server release. This transition represents the most significant shift in Windows enterprise authentication since Kerberos replaced NTLM as the default protocol in Windows 2000. Organizations that delay migration face escalating risk: at least 10 actively exploited NTLM CVEs were disclosed in 2024-2025 alone, and identity-based attacks became the leading intrusion vector in 2024, accounting for **30% of all breaches** according to IBM X-Force. Meanwhile, Kerberos itself faces a parallel crisis. Newly discovered attack techniques like BadSuccessor and Golden dMSA exploit the very features Microsoft designed to replace legacy weaknesses, while Kerberoasting remains the dominant credential-theft technique across enterprise environments.

---

## Key Findings

1. **Microsoft formally deprecated all NTLM versions in June 2024 and published a three-phase deprecation roadmap on January 29, 2026.** NTLMv1 has been fully removed from Windows 11 24H2 and Windows Server 2025. NTLMv2 will be disabled by default in the next major Windows Server LTSC release. Complete removal has no announced date.

2. **At least 10 actively exploited NTLM CVEs were disclosed in 2024-2025, including two CVSS 9.8 vulnerabilities.** The patch-bypass-repatch cycle around CVE-2025-24054 demonstrates that NTLM's architectural weaknesses cannot be incrementally fixed. Cymulate bypassed Microsoft's patches twice, achieving zero-click credential leakage on fully patched systems.

3. **Kerberoasting activity increased 583% year-over-year** (CrowdStrike), and new attack techniques targeting Windows Server 2025 features designed to replace NTLM (BadSuccessor, Golden dMSA) show that Kerberos itself requires significant hardening.

4. **64% of Active Directory user accounts still regularly authenticate with NTLM** (Silverfort), and Gartner estimates over 50% of organizations actively use it. Enterprise migration typically requires 18-22 months. The gap between deprecation intent and operational reality is substantial.

5. **The passwordless authentication market has reached a tipping point.** FIDO2 passkeys achieve a 93% login success rate versus 63% for traditional authentication. Microsoft made passkeys the default sign-in for all new accounts in May 2025. The market is valued at $18.8-24.1 billion and projected to reach $55-90 billion by 2030-2035.

6. **Nation-state actors weaponize NTLM vulnerabilities within days of disclosure.** CVE-2025-24054 was exploited eight days after patch release, with campaigns targeting Polish and Romanian government institutions traced to infrastructure previously linked to APT28 (Fancy Bear). Volt Typhoon, Scattered Spider, and Wizard Spider all incorporate NTLM exploitation into their toolkits.

---

## Forward-Looking Assumptions

1. **By mid-2027, Microsoft will disable NTLMv2 by default in the next Windows Server LTSC release,** forcing organizations that have not migrated into explicit re-enablement through policy. Organizations without NTLM audit data by Q4 2026 will face emergency remediation timelines.

2. **Through 2028, Kerberoasting will remain the dominant Active Directory credential-theft technique,** despite RC4 deprecation efforts. Enterprises with mixed Windows Server 2019/2025 environments will encounter encryption type mismatches that slow AES-only enforcement.

3. **By 2027, at least one major breach attributed to a BadSuccessor or Golden dMSA attack will force Microsoft to reclassify dMSA attack surface from "moderate" to "critical" severity.** Akamai found 91% of environments had non-admin users with sufficient permissions to execute BadSuccessor. The attack surface is too broad for the current severity classification to hold.

4. **By 2029, FIDO2/passkeys will be the default enterprise authentication method for over 50% of Fortune 500 companies.** The combination of Microsoft, Google, and Apple platform support, 87% enterprise deployment or planning rates (FIDO Alliance), and regulatory pressure from CISA and NIST will drive adoption past the tipping point.

5. **Through 2027, the most common cause of NTLM migration failure will be hidden NTLM fallback from misconfigured SPNs and IP-based access patterns,** not legacy application hardcoding. Phase 1 auditing will reveal NTLM dependencies that organizations did not know existed, extending migration timelines beyond initial estimates.

---

## Analysis

### 1. Microsoft's Three-Phase Deprecation Roadmap

Microsoft formally deprecated all NTLM versions (LANMAN, NTLMv1, and NTLMv2) in **June 2024**, adding them to the Windows Deprecated Features list. NTLMv1 was not just deprecated but **fully removed** from Windows 11 24H2 and Windows Server 2025. NTLMv2 remains functional but receives no active development. On January 29, 2026, Microsoft published a definitive roadmap with three phases.

**Phase 1 (available now)** delivers enhanced NTLM auditing in Windows Server 2025 and Windows 11 24H2. New event IDs (4020-4033) capture which accounts use NTLM, which processes trigger it, why Kerberos failed, and the NTLM version negotiated. This visibility layer is the critical first step. Most organizations cannot enumerate their NTLM dependencies without it.

**Phase 2 (H2 2026)** addresses the technical barriers that force NTLM fallback. Two new capabilities ship: **IAKerb** (Initial and Pass Through Authentication Using Kerberos), which enables Kerberos authentication without direct domain controller connectivity, and **Local KDC**, which provides Kerberos for local account authentication. Microsoft estimates these cover roughly 5% of remaining NTLM usage, specifically the hardest cases involving workgroup-joined systems, IP-based access, and disconnected scenarios.

**Phase 3 (next major Windows Server LTSC release)** will **disable network NTLM by default**. Applications requiring NTLM must explicitly re-enable it through policy. Microsoft has been careful to clarify: *"Disabling NTLM by default does not mean completely removing NTLM from Windows yet."* Complete removal has no announced date. The `BlockNTLMv1SSO` registry key will flip from audit to enforce mode by **October 2026**, and **RC4 encryption for Kerberos**, the cryptographic weakness enabling Kerberoasting, will be disabled by default for new domains in Q1 2026, with broader enforcement by mid-2026.

### 2. Kerberos Under Siege: Old Attacks Persist, New Ones Emerge

While Microsoft positions Kerberos as NTLM's successor, the protocol faces its own expanding attack surface. **Kerberoasting** remains the single most impactful Active Directory attack technique. CrowdStrike documented a **583% year-over-year increase** in Kerberoasting activity, and the Ascension Health ransomware breach in May 2024, which exposed 5.6 million patient records, was enabled by RC4 support in Kerberos. That breach prompted a U.S. Senator to demand an FTC investigation into Microsoft's security defaults.

The traditional attack taxonomy (Golden Tickets, Silver Tickets, Pass-the-Ticket, AS-REP Roasting) remains fully relevant. But researchers have introduced increasingly stealthy variants that challenge conventional detection:

- **Diamond Tickets** modify a legitimately issued TGT by decrypting it with the KRBTGT key, altering the PAC to add privileged group memberships, and re-encrypting. Because the ticket has a corresponding AS-REQ in KDC logs, it evades detections that look for forged tickets without authentication history.
- **Sapphire Tickets** go further, replacing the PAC entirely with a legitimate PAC obtained through S4U2Self+U2U extensions. The resulting ticket contains no forged data whatsoever, making detection through PAC inspection effectively impossible.
- **BadSuccessor** (disclosed by Akamai, April 2025) exploits Delegated Managed Service Accounts (dMSA), a Windows Server 2025 feature designed to mitigate Kerberoasting. An attacker with CreateChild permissions on any OU can create a dMSA that impersonates a Domain Admin. Akamai found **91% of environments** had non-admin users with sufficient permissions to execute this attack. Microsoft classified it as "moderate severity" and released a patch in August 2025.
- **Golden dMSA** (Semperis, July 2025) enables an attacker who obtains the KDS root key to brute-force valid dMSA passwords offline with only ~1,024 possible combinations, providing persistent access to all managed service accounts in the forest with no expiration.

The most critical Kerberos CVE of this period was **CVE-2024-43639 (CVSS 9.8)**, an unauthenticated remote code execution vulnerability in the Windows KDC Proxy caused by an integer overflow. It required no user interaction and affected Windows Server 2012 through 2025. Microsoft's **PAC validation enforcement**, completed in April 2025 after a multi-year rollout, represents the most significant defensive improvement, making Golden and Silver Ticket attacks harder to execute undetected.

### 3. NTLM's 2024-2025 CVE Crisis

The volume and severity of NTLM vulnerabilities disclosed in 2024-2025 has been extraordinary, demonstrating that NTLM's fundamental design cannot be patched into safety.

| CVE | CVSS | Type | Exploited in Wild | Key Detail |
|---|---|---|---|---|
| CVE-2024-21410 | 9.8 | Exchange NTLM relay | Yes | Prompted default EPA enablement |
| CVE-2024-43451 | 6.5 | NTLMv2 hash disclosure (zero-day) | Yes | Used by Russian-linked UAC-0194, BlindEagle, Head Mare |
| CVE-2025-21311 | **9.8** | NTLMv1 privilege escalation | Automatable | CISA flagged "total technical impact" |
| CVE-2025-24054 | 6.5 | Hash disclosure via .library-ms | Yes | Exploited 8 days after patch; CISA KEV |
| CVE-2025-33073 | High | NTLM reflection to SYSTEM | PoC available | Any domain user to SYSTEM on hosts without SMB signing |
| CVE-2025-59214 | Medium | Third bypass of hash disclosure patch | Demonstrated | Zero-click on fully patched systems |

**CVE-2025-24054** illustrates the problem's urgency. Patched on March 11, 2025, it was exploited in the wild by March 19, just **eight days** later. Check Point Research documented approximately 10 campaigns by March 25, targeting Polish and Romanian government institutions. SMB hash-collection servers were traced to Russia, Bulgaria, Netherlands, Australia, and Turkey, with one IP previously linked to **APT28 (Fancy Bear)**. CISA added it to the Known Exploited Vulnerabilities catalog with a mandatory remediation deadline.

Cymulate Research Labs discovered that Microsoft's patches for CVE-2025-24054 could be **bypassed twice** (CVE-2025-50154 and CVE-2025-59214), with the latter achieving zero-click NTLM credential leakage on fully patched systems. This pattern of patch-bypass-repatch underscores that NTLM's architectural weaknesses cannot be incrementally fixed. They require protocol elimination.

Nation-state actors and ransomware groups actively weaponize these weaknesses. **Volt Typhoon**, **APT28**, **Scattered Spider**, and **Wizard Spider** all incorporate NTLM exploitation into their toolkits. **90% of ransomware breaches** involve RDP abuse (Sophos), and groups like ALPHV/BlackCat, Akira, and RansomHub routinely leverage NTLM-based lateral movement to escalate from initial access to domain compromise.

### 4. Enterprise Migration Barriers

Despite the clear threat, enterprise NTLM elimination remains a significant operational challenge. Silverfort's research found that **64% of Active Directory user accounts regularly authenticate with NTLM**, and Gartner estimates over **50% of organizations** still actively use it.

The primary obstacles are well-documented. Legacy applications (ERP systems, HR platforms, and industrial control software built before the 2000s) often hardcode NTLM with no Kerberos support. Third-party firmware in printers, network devices, and IoT equipment frequently embeds NTLMv1 with no upgrade path. Hidden NTLM fallback occurs when Kerberos fails silently due to misconfigured SPNs, IP-based access patterns, or missing DNS entries. Cross-forest trust scenarios and Exchange hybrid migrations add further complexity. Mandiant's January 2026 release of **8.6 terabytes of NTLMv1 rainbow tables**, enabling hash recovery in under 12 hours on $600 hardware, was a deliberate forcing function designed to eliminate any remaining justification for NTLMv1 retention.

Organizations following Microsoft's recommended migration path typically plan **18-22 months** for full NTLM elimination. The most heavily affected industries include **manufacturing** (50% of observed NTLM exploitation targets per Kaspersky telemetry), **healthcare** (the Ascension breach being the most prominent case), **government** (CISA assessments identify credential access as the most prevalent attack against federal agencies), and **financial services** (where strict compliance requirements both motivate and complicate migration timelines).

### 5. Vendor Landscape

The NTLM deprecation has catalyzed a competitive vendor landscape spanning detection, migration, governance, and replacement.

**Microsoft** is driving the transition most aggressively, with Entra ID as the centerpiece. Microsoft Entra Private Access (GA July 2024) acts as an OAuth/OIDC-to-Kerberos bridge, layering Conditional Access and MFA on legacy NTLM/Kerberos applications without code changes. Entra Kerberos (Cloud Kerberos Trust) turns Entra ID into a cloud-based KDC, enabling passwordless SSO to on-premises resources and is now the recommended deployment model for Windows Hello for Business. Windows Server 2025 ships with **EPA enabled by default** for AD CS and LDAP, **Credential Guard enabled by default**, and mandatory SMB signing.

**Silverfort** has emerged as the most prominent vendor specifically addressing NTLM security, offering the only solution that extends MFA to NTLM and Kerberos authentications without agents. Their research discovered that on-premises applications can bypass the Group Policy designed to block NTLMv1, creating what they called a "false sense of protection." Gartner's May 2025 report "A Well-Run Active Directory Requires Strong Identity Controls" named Silverfort as an example vendor in three categories.

**CrowdStrike's Falcon Identity Protection** performs real-time inspection of NTLM, Kerberos, and LDAP traffic, with specialized detection for relay attacks, Kerberoasting, and Golden/Silver Ticket usage. Their acquisition of Preempt Security in 2020, whose researchers discovered critical NTLM bypass vulnerabilities, underpins deep protocol expertise.

**Semperis** focuses on AD resilience and recovery, with Directory Services Protector monitoring for NTLM-related indicators and a new Service Account Protection Essential (August 2025) specifically targeting Kerberoasting-vulnerable service accounts. Their researchers discovered the Golden dMSA attack and coordinated multiple CVE disclosures with Microsoft.

**Cisco Duo** made a significant competitive entry in 2025-2026, extending MFA directly to all Active Directory authentications, including CLI and legacy applications using Kerberos and NTLM. Cisco Identity Intelligence provides deep posture management, and Cisco Talos found nearly half of identity-based attacks in 2024 focused on Active Directory.

**Delinea** explicitly added Kerberos authentication support to Connection Manager in Q1 2025, noting it "mitigates risks associated with NTLM, including Pass-the-Hash, DCSync, NTLM relay." **Okta's** Agentless Desktop SSO explicitly requires Kerberos tickets; NTLM tokens cause authentication failure by design. **Ping Identity** and **SailPoint** operate at the federation and governance layers respectively, providing standards-based SSO and identity lifecycle management that inherently bypass NTLM.

### 6. Modern Alternatives Reaching Enterprise Maturity

The authentication stack replacing NTLM and password-based Kerberos is coalescing around three pillars: passwordless credentials, cloud identity platforms, and Zero Trust architecture.

**FIDO2 and passkeys reached a tipping point in 2025.** The FIDO Alliance reports that 87% of US and UK enterprises are deploying or planning passkey deployment for employee sign-ins. Passkeys achieve a **93% login success rate** versus 63% for traditional authentication. Microsoft made passkeys the default sign-in for all new Microsoft accounts in May 2025, driving 120% growth in passkey authentications. Okta's data shows phishing-resistant passwordless authentication grew 63% year-over-year, and workforce MFA adoption reached 70%.

**Certificate-based authentication** serves regulated environments requiring digital signatures, email encryption, and high-assurance identity proofing. Microsoft Entra CBA is now generally available, enabling X.509 certificate authentication directly to Entra ID without AD FS and classified as phishing-resistant MFA by both Microsoft and CISA. Federal agencies using PIV/CAC smart cards can authenticate directly to cloud resources, removing a lateral movement path through Active Directory.

**Zero Trust architecture** provides the strategic framework driving NTLM elimination. CISA's Zero Trust Maturity Model v2.0 requires phishing-resistant MFA at the "Advanced" level and continuous identity validation at the "Optimal" level, both incompatible with NTLM's implicit trust model. **OMB M-22-09** mandates federal agencies achieve Zero Trust objectives including phishing-resistant MFA. Internationally, the EU's NIS2 Directive, eIDAS 2.0 (requiring digital identity wallets by 2026), and PCI DSS 4.0 all strengthen authentication requirements. Eighty-one percent of companies are pursuing some form of Zero Trust strategy.

---

## Recommendations for Security and Risk Management Leaders

1. **Deploy Phase 1 NTLM auditing immediately.** Windows Server 2025 and Windows 11 24H2 provide new event IDs (4020-4033) that map all NTLM dependencies. Without this data, migration planning is guesswork. Set a 90-day deadline for complete NTLM dependency inventory.

2. **Enforce NTLMv2-only and block NTLMv1 now.** Set LmCompatibilityLevel=5 across all systems. NTLMv1 is already removed from current operating systems. Mandiant's rainbow tables make any remaining NTLMv1 usage an immediate compromise risk.

3. **Begin RC4 deprecation for Kerberos as the single highest-impact defensive action.** Kerberoasting depends on RC4. Disable RC4 for new domains immediately; plan AES-only enforcement for existing domains by mid-2026. Test thoroughly in mixed Windows Server 2019/2025 environments where encryption type mismatches cause authentication failures.

4. **Enable EPA, SMB signing, and Credential Guard on all servers.** These are the three controls that most effectively mitigate NTLM relay, pass-the-hash, and credential theft. Windows Server 2025 enables them by default; backport to older systems through Group Policy.

5. **Block NTLM progressively, starting with domain controllers and certificate authorities.** Add privileged accounts to the Protected Users group. Use the Negotiate package instead of explicit NTLM calls (often a one-line code change). Plan 18-22 months for full domain-wide NTLM blocking.

6. **Audit dMSA permissions immediately.** Akamai found 91% of environments have non-admin users with sufficient permissions to execute the BadSuccessor attack. Restrict CreateChild permissions on OUs and monitor dMSA creation events until Microsoft's patch is widely deployed.

7. **Invest in phishing-resistant authentication infrastructure.** FIDO2 passkeys and certificate-based authentication through Entra ID are the long-term replacements for password-based protocols. Begin pilot deployments now; align with NIST SP 800-63B-4 requirements for AAL3 hardware-based authenticators.

8. **Block outbound SMB (TCP 445) at the network perimeter.** This prevents NTLM hash exfiltration to attacker-controlled servers, the technique used in CVE-2025-24054 campaigns. This is a low-effort, high-impact network control.

---

## Market Outlook

The NTLM deprecation is creating distinct market dynamics across three vendor categories. Identity protection vendors (Silverfort, CrowdStrike, Semperis) are best positioned in the near term because they address the immediate operational challenge: detecting and controlling NTLM usage during the multi-year migration period. These vendors have a finite window of relevance tied to NTLM's lifecycle, but the 18-22 month migration timeline and the long tail of legacy environments means that window extends through at least 2029.

Microsoft holds the strongest structural position. Entra ID, IAKerb, Local KDC, and the Windows Server 2025 security defaults form a coherent stack that addresses both the deprecation path and the replacement architecture. Organizations heavily invested in Microsoft infrastructure will find the migration path most natural, though multi-cloud and hybrid environments will require supplementary tooling.

The passwordless authentication market ($18.8-24.1 billion, projected to $55-90 billion by 2030-2035) reflects the scale of the broader transition away from password-based protocols. FIDO2/passkey vendors, cloud identity platforms, and Zero Trust architecture providers are the long-term beneficiaries. The organizations that treat NTLM elimination as a compliance checkbox rather than an architectural shift will find themselves repeatedly patching symptoms while the underlying protocol remains a liability.

The honest assessment: NTLM elimination is operationally harder than any vendor marketing suggests, and Kerberos hardening introduces its own risks. But the alternative, maintaining a protocol with a demonstrated pattern of unpatchable vulnerabilities while nation-state actors exploit them within days, is no longer defensible.

---

## Key Sources

**Microsoft documentation:** NTLM Deprecation Roadmap (January 29, 2026); Windows Server 2025 Security Defaults; IAKerb and Local KDC announcements; Entra Private Access GA (July 2024); Entra Kerberos Cloud Trust documentation.

**Vulnerability research:** Check Point Research CVE-2025-24054 analysis (March 2025); Cymulate Research Labs CVE-2025-50154 and CVE-2025-59214 bypass disclosures; Akamai BadSuccessor disclosure (April 2025); Semperis Golden dMSA disclosure (July 2025); Microsoft CVE-2024-43639 advisory.

**Threat intelligence:** CrowdStrike 2025 Global Threat Report (583% Kerberoasting increase); IBM X-Force 2024 Threat Intelligence Index (30% identity-based breaches); Sophos Active Adversary Report (90% RDP abuse in ransomware); Mandiant NTLMv1 rainbow tables release (January 2026); CISA Known Exploited Vulnerabilities catalog.

**Industry data:** Silverfort NTLM usage research (64% of AD accounts); Gartner estimates (50%+ organizations using NTLM); FIDO Alliance enterprise deployment survey (87% deploying or planning); Kaspersky NTLM exploitation telemetry; Ascension Health breach disclosures (May 2024).

**Government guidance:** Five Eyes joint advisory "Detecting and Mitigating Active Directory Compromises" (September 2024, updated January 2025); NIST SP 800-63B-4 (July 2025); CISA Zero Trust Maturity Model v2.0; CISA phishing-resistant MFA fact sheet; OMB M-22-09.

**Vendor sources:** Silverfort NTLMv1 bypass research; CrowdStrike Falcon Identity Protection; Semperis Directory Services Protector; Cisco Duo AD authentication extension; Delinea Connection Manager Kerberos support (Q1 2025); Gartner "A Well-Run Active Directory Requires Strong Identity Controls" (May 2025).
