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

Microsoft is dismantling NTLM, the 30-year-old authentication protocol still used by 64% of Active Directory accounts, through a three-phase deprecation plan that will disable it by default in the next major Windows Server release. This transition represents the most significant shift in Windows enterprise authentication since Kerberos replaced NTLM as the default protocol in Windows 2000. Organizations that delay migration face escalating risk: at least 10 actively exploited NTLM CVEs were disclosed in 2024-2025 alone, and identity-based attacks became the leading intrusion vector in 2024, accounting for **30% of all breaches** according to IBM X-Force. Meanwhile, Kerberos itself faces a parallel crisis. Newly discovered attack techniques like BadSuccessor and Golden dMSA exploit the very features Microsoft designed to replace legacy weaknesses, while Kerberoasting remains the dominant credential-theft technique across enterprise environments. This report provides a comprehensive assessment of both protocols' security posture, the threat landscape, vendor responses, and the modern alternatives enterprises must adopt.

---

## Microsoft's three-phase plan to kill NTLM

Microsoft formally deprecated all NTLM versions (LANMAN, NTLMv1, and NTLMv2) in **June 2024**, adding them to the Windows Deprecated Features list. NTLMv1 was not just deprecated but **fully removed** from Windows 11 24H2 and Windows Server 2025. NTLMv2 remains functional but receives no active development. On January 29, 2026, Microsoft published a definitive roadmap with three phases.

**Phase 1 (available now)** delivers enhanced NTLM auditing in Windows Server 2025 and Windows 11 24H2. New event IDs (4020-4033) capture which accounts use NTLM, which processes trigger it, why Kerberos failed, and the NTLM version negotiated. This visibility layer is the critical first step. Most organizations cannot enumerate their NTLM dependencies without it.

**Phase 2 (H2 2026)** addresses the technical barriers that force NTLM fallback. Two new capabilities ship: **IAKerb** (Initial and Pass Through Authentication Using Kerberos), which enables Kerberos authentication without direct domain controller connectivity, and **Local KDC**, which provides Kerberos for local account authentication. Microsoft estimates these cover roughly 5% of remaining NTLM usage, specifically the hardest cases involving workgroup-joined systems, IP-based access, and disconnected scenarios.

**Phase 3 (next major Windows Server LTSC release)** will **disable network NTLM by default**. Applications requiring NTLM must explicitly re-enable it through policy. Microsoft has been careful to clarify: *"Disabling NTLM by default does not mean completely removing NTLM from Windows yet."* Complete removal has no announced date. Meanwhile, the `BlockNTLMv1SSO` registry key will flip from audit to enforce mode by **October 2026**, and **RC4 encryption for Kerberos**, the cryptographic weakness enabling Kerberoasting, will be disabled by default for new domains in Q1 2026, with broader enforcement by mid-2026.

---

## Kerberos under siege: old attacks persist, new ones emerge

While Microsoft positions Kerberos as NTLM's successor, the protocol faces its own expanding attack surface. **Kerberoasting** remains the single most impactful Active Directory attack technique. CrowdStrike documented a **583% year-over-year increase** in Kerberoasting activity, and the Ascension Health ransomware breach in May 2024, which exposed 5.6 million patient records, was enabled by RC4 support in Kerberos. That breach prompted a U.S. Senator to demand an FTC investigation into Microsoft's security defaults.

The traditional attack taxonomy (Golden Tickets, Silver Tickets, Pass-the-Ticket, AS-REP Roasting) remains fully relevant. But researchers have introduced increasingly stealthy variants that challenge conventional detection:

- **Diamond Tickets** modify a legitimately issued TGT by decrypting it with the KRBTGT key, altering the PAC to add privileged group memberships, and re-encrypting. Because the ticket has a corresponding AS-REQ in KDC logs, it evades detections that look for forged tickets without authentication history.
- **Sapphire Tickets** go further, replacing the PAC entirely with a legitimate PAC obtained through S4U2Self+U2U extensions. The resulting ticket contains no forged data whatsoever, making detection through PAC inspection effectively impossible.
- **BadSuccessor** (disclosed by Akamai, April 2025) exploits Delegated Managed Service Accounts (dMSA), a Windows Server 2025 feature designed to mitigate Kerberoasting. An attacker with CreateChild permissions on any OU can create a dMSA that impersonates a Domain Admin. Akamai found **91% of environments** had non-admin users with sufficient permissions to execute this attack. Microsoft classified it as "moderate severity" and released a patch in August 2025.
- **Golden dMSA** (Semperis, July 2025) enables an attacker who obtains the KDS root key to brute-force valid dMSA passwords offline with only ~1,024 possible combinations, providing persistent access to all managed service accounts in the forest with no expiration.

The most critical Kerberos CVE of this period was **CVE-2024-43639 (CVSS 9.8)**, an unauthenticated remote code execution vulnerability in the Windows KDC Proxy caused by an integer overflow. It required no user interaction and affected Windows Server 2012 through 2025. Microsoft's **PAC validation enforcement**, completed in April 2025 after a multi-year rollout, represents the most significant defensive improvement, making Golden and Silver Ticket attacks harder to execute undetected.

---

## NTLM's 2024-2025 CVE crisis accelerated deprecation

The volume and severity of NTLM vulnerabilities disclosed in 2024-2025 has been extraordinary, demonstrating that NTLM's fundamental design cannot be patched into safety.

| CVE | CVSS | Type | Exploited in Wild | Key Detail |
|---|---|---|---|---|
| CVE-2024-21410 | 9.8 | Exchange NTLM relay | Yes | Prompted default EPA enablement |
| CVE-2024-43451 | 6.5 | NTLMv2 hash disclosure (zero-day) | Yes | Used by Russian-linked UAC-0194, BlindEagle, Head Mare |
| CVE-2025-21311 | **9.8** | NTLMv1 privilege escalation | Automatable | CISA flagged "total technical impact" |
| CVE-2025-24054 | 6.5 | Hash disclosure via .library-ms | Yes | Exploited 8 days after patch; CISA KEV |
| CVE-2025-33073 | High | NTLM reflection to SYSTEM | PoC available | Any domain user to SYSTEM on hosts without SMB signing |
| CVE-2025-59214 | Medium | Third bypass of hash disclosure patch | Demonstrated | Zero-click on fully patched systems |

**CVE-2025-24054** is a good illustration of the problem's urgency. Patched on March 11, 2025, it was exploited in the wild by March 19, just **eight days** later. Check Point Research documented approximately 10 campaigns by March 25, targeting Polish and Romanian government institutions. SMB hash-collection servers were traced to Russia, Bulgaria, Netherlands, Australia, and Turkey, with one IP previously linked to **APT28 (Fancy Bear)**. CISA added it to the Known Exploited Vulnerabilities catalog with a mandatory remediation deadline.

Perhaps most alarming: Cymulate Research Labs discovered that Microsoft's patches for CVE-2025-24054 could be **bypassed twice** (CVE-2025-50154 and CVE-2025-59214), with the latter achieving zero-click NTLM credential leakage on fully patched systems. This pattern of patch-bypass-repatch underscores that NTLM's architectural weaknesses cannot be incrementally fixed. They require protocol elimination.

Nation-state actors and ransomware groups actively weaponize these weaknesses. **Volt Typhoon**, **APT28**, **Scattered Spider**, and **Wizard Spider** all incorporate NTLM exploitation into their toolkits. **90% of ransomware breaches** involve RDP abuse (Sophos), and groups like ALPHV/BlackCat, Akira, and RansomHub routinely leverage NTLM-based lateral movement to escalate from initial access to domain compromise.

---

## Enterprise migration: urgent but painfully slow

Despite the clear threat, enterprise NTLM elimination remains a significant operational challenge. Silverfort's research found that **64% of Active Directory user accounts regularly authenticate with NTLM**, and Gartner estimates over **50% of organizations** still actively use it. The gap between deprecation intent and operational reality is substantial.

The primary obstacles are well-documented. Legacy applications (ERP systems, HR platforms, and industrial control software built before the 2000s) often hardcode NTLM with no Kerberos support. Third-party firmware in printers, network devices, and IoT equipment frequently embeds NTLMv1 with no upgrade path. Hidden NTLM fallback occurs when Kerberos fails silently due to misconfigured SPNs, IP-based access patterns, or missing DNS entries. Cross-forest trust scenarios and Exchange hybrid migrations add further complexity. Mandiant's January 2026 release of **8.6 terabytes of NTLMv1 rainbow tables**, enabling hash recovery in under 12 hours on $600 hardware, was a deliberate forcing function designed to eliminate any remaining justification for NTLMv1 retention.

Organizations following Microsoft's recommended migration path typically plan **18-22 months** for full NTLM elimination. The process begins with deploying Phase 1 auditing to map all NTLM dependencies, then systematically replacing explicit NTLM calls with the Negotiate package (often a one-line code change), fixing SPN configurations, enforcing EPA and SMB signing, enabling Credential Guard, and progressively blocking NTLM: first on domain controllers and certificate authorities, then on critical servers, and finally domain-wide.

The most heavily affected industries include **manufacturing** (50% of observed NTLM exploitation targets per Kaspersky telemetry), **healthcare** (the Ascension breach being the most prominent case), **government** (CISA assessments identify credential access as the most prevalent attack against federal agencies), and **financial services** (where strict compliance requirements both motivate and complicate migration timelines).

---

## How vendors are responding to the authentication transition

The NTLM deprecation has catalyzed a competitive vendor landscape spanning detection, migration, governance, and replacement.

**Microsoft** is driving the transition most aggressively, with Entra ID as the centerpiece. Microsoft Entra Private Access (GA July 2024) acts as an OAuth/OIDC-to-Kerberos bridge, layering Conditional Access and MFA on legacy NTLM/Kerberos applications without code changes. Entra Kerberos (Cloud Kerberos Trust) turns Entra ID into a cloud-based KDC, enabling passwordless SSO to on-premises resources and is now the recommended deployment model for Windows Hello for Business. Windows Server 2025 ships with **EPA enabled by default** for AD CS and LDAP, **Credential Guard enabled by default**, and mandatory SMB signing. Microsoft also provides dedicated migration tooling: PowerShell scripts for auditing, the Protected Users group for forcing Kerberos-only authentication, and a dedicated email (ntlm@microsoft.com) for edge-case reporting.

**Silverfort** has emerged as the most prominent vendor specifically addressing NTLM security, offering the only solution that extends MFA to NTLM and Kerberos authentications without agents. Their research discovered that on-premises applications can bypass the Group Policy designed to block NTLMv1, creating what they called a "false sense of protection." Gartner's May 2025 report "A Well-Run Active Directory Requires Strong Identity Controls" named Silverfort as an example vendor in three categories.

**CrowdStrike's Falcon Identity Protection** performs real-time inspection of NTLM, Kerberos, and LDAP traffic, with specialized detection for relay attacks, Kerberoasting, and Golden/Silver Ticket usage. Their acquisition of Preempt Security in 2020, whose researchers discovered critical NTLM bypass vulnerabilities, underpins deep protocol expertise.

**Semperis** focuses on AD resilience and recovery, with Directory Services Protector monitoring for NTLM-related indicators and a new Service Account Protection Essential (August 2025) specifically targeting Kerberoasting-vulnerable service accounts. Their researchers discovered the Golden dMSA attack and coordinated multiple CVE disclosures with Microsoft.

**Cisco Duo** made a significant competitive entry in 2025-2026, extending MFA directly to all Active Directory authentications, including CLI and legacy applications using Kerberos and NTLM. Cisco Identity Intelligence provides deep posture management, and Cisco Talos found nearly half of identity-based attacks in 2024 focused on Active Directory.

**Delinea** explicitly added Kerberos authentication support to Connection Manager in Q1 2025, noting it "mitigates risks associated with NTLM, including Pass-the-Hash, DCSync, NTLM relay." **Okta's** Agentless Desktop SSO explicitly requires Kerberos tickets; NTLM tokens cause authentication failure by design. **Ping Identity** and **SailPoint** operate at the federation and governance layers respectively, providing standards-based SSO and identity lifecycle management that inherently bypass NTLM.

---

## Modern alternatives reaching enterprise maturity

The authentication stack replacing NTLM and password-based Kerberos is coalescing around three pillars: passwordless credentials, cloud identity platforms, and Zero Trust architecture.

**FIDO2 and passkeys reached a tipping point in 2025.** The FIDO Alliance reports that 87% of US and UK enterprises are deploying or planning passkey deployment for employee sign-ins. Passkeys achieve a **93% login success rate** versus 63% for traditional authentication. Microsoft made passkeys the default sign-in for all new Microsoft accounts in May 2025, driving 120% growth in passkey authentications. Okta's data shows phishing-resistant passwordless authentication grew 63% year-over-year, and workforce MFA adoption reached 70%. The passwordless authentication market is valued at **$18.8-24.1 billion** and projected to reach $55-90 billion by 2030-2035.

**Certificate-based authentication** serves regulated environments requiring digital signatures, email encryption, and high-assurance identity proofing. Microsoft Entra CBA is now generally available, enabling X.509 certificate authentication directly to Entra ID without AD FS and classified as phishing-resistant MFA by both Microsoft and CISA. Federal agencies using PIV/CAC smart cards can authenticate directly to cloud resources, removing a lateral movement path through Active Directory. IDManagement.gov published a detailed implementation guide for federal agencies.

**Zero Trust architecture** provides the strategic framework driving NTLM elimination. CISA's Zero Trust Maturity Model v2.0 requires phishing-resistant MFA at the "Advanced" level and continuous identity validation at the "Optimal" level, both incompatible with NTLM's implicit trust model. **OMB M-22-09** mandates federal agencies achieve Zero Trust objectives including phishing-resistant MFA. Internationally, the EU's NIS2 Directive, eIDAS 2.0 (requiring digital identity wallets by 2026), and PCI DSS 4.0 all strengthen authentication requirements. Eighty-one percent of companies are pursuing some form of Zero Trust strategy.

---

## What the Five Eyes, CISA, and NIST recommend now

The **Five Eyes joint advisory** "Detecting and Mitigating Active Directory Compromises" (September 2024, updated January 2025), authored by CISA, NSA, ASD, and partner agencies, provides the most authoritative operational guidance. It covers 17 attack techniques targeting Active Directory and recommends: implementing Microsoft's Enterprise Access Model (Tier 0/1/2), converting service accounts to Group Managed Service Accounts, ensuring Kerberos pre-authentication on all accounts, disabling NTLM and SMBv1, deploying phishing-resistant MFA for Tier-0 accounts, using Privileged Access Workstations, and rotating the KRBTGT password at least every 180 days.

**NIST SP 800-63B-4** (finalized July 2025) defines Authentication Assurance Level 3 as requiring hardware-based, phishing-resistant authenticators, implicitly requiring organizations to move beyond NTLM and password-based Kerberos. CISA's phishing-resistant MFA fact sheet names **FIDO2/WebAuthn and PKI (PIV/CAC)** as the "gold standard," explicitly warning that OTP and push notifications remain susceptible to phishing, SIM-swap, and prompt-bombing attacks.

The consensus migration roadmap across Microsoft, CISA, and leading security vendors follows a clear sequence:

- **Immediate (0-3 months):** Patch all NTLM/Kerberos CVEs. Enforce NTLMv2-only via LmCompatibilityLevel=5. Enable SMB signing and EPA on all servers. Block outbound SMB (TCP 445) at the network perimeter. Deploy Credential Guard.
- **Short-term (3-6 months):** Deploy enhanced NTLM auditing. Identify all NTLM dependencies. Disable LLMNR and NetBIOS over TCP/IP. Convert service accounts to gMSAs. Begin disabling RC4 for Kerberos.
- **Medium-term (6-12 months):** Block NTLM on domain controllers, certificate authorities, and critical servers. Add privileged accounts to the Protected Users group. Implement tiered administration with Privileged Access Workstations.
- **Long-term (12-24 months):** Block NTLM domain-wide. Enforce AES-only Kerberos encryption. Align full removal with Microsoft's deprecation timeline. Deploy phishing-resistant MFA universally.

---

## Conclusion

The NTLM deprecation is not a theoretical future event. It is an active, multi-year process with NTLMv1 already removed from current operating systems and NTLMv2's default disablement approaching. Organizations that treat this as a distant concern face compounding risk: the cadence of actively exploited NTLM CVEs is accelerating, patch bypasses demonstrate the protocol cannot be incrementally secured, and nation-state actors routinely weaponize NTLM weaknesses within days of disclosure.

Kerberos, while the designated successor for on-premises authentication, requires its own hardening revolution. **RC4 deprecation is the single most impactful defensive action** organizations can take against Kerberoasting, but the BadSuccessor and Golden dMSA attacks reveal that new features designed to improve security can introduce novel attack surfaces. Windows Server 2025 mixed-environment deployments demand careful testing, as encryption type mismatches between old and new domain controllers have caused widespread authentication failures.

The organizations best positioned for this transition are those investing now in three capabilities: **comprehensive NTLM visibility** (using Phase 1 auditing tools), **modern credential infrastructure** (FIDO2, passkeys, certificate-based authentication through Entra ID), and **Zero Trust architecture** that eliminates implicit trust at the protocol level. The passwordless authentication market's projected growth to $55-90 billion by 2030 reflects the scale of this transition. Every enterprise security roadmap should treat legacy authentication protocol elimination not as a compliance checkbox but as a foundational prerequisite for defending against the current threat landscape.
