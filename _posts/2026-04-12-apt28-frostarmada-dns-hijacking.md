---
title: "Operation FrostArmada: APT28 Compromises 18,000 SOHO Routers for Global DNS Hijacking and Credential Theft"
date: 2026-04-12
categories: [APT, Nation-State, Credential Theft]
tags: [APT28, DNS Hijacking, Forest Blizzard, GRU, Russia, SOHO Routers, AitM, Microsoft 365, FrostArmada, PRISMEX]
image:
  path: /assets/img/blog/2026-04-12/thumbnail-frostarmada.png
---

## Overview

**Who:** APT28 (also tracked as Forest Blizzard, Fancy Bear, Pawn Storm, Sednit, STRONTIUM, and Storm-2754) is the offensive cyber unit of Russia's GRU Military Unit 26165, the 85th Main Special Service Center (GTsSS). The group has operated continuously since at least 2004 and has been formally attributed to the Russian military by the United States, United Kingdom, and allied governments through multiple criminal indictments. Prior campaigns of documented significance include the 2015 compromise of the German Bundestag, the 2016 breach of the Democratic National Committee, and sustained targeting of NATO member institutions, international election infrastructure, and organizations including WADA and OPCW. Within the FrostArmada campaign, the sub-cluster responsible for the router exploitation phase has been separately designated Storm-2754 by Microsoft.

**What:** A large-scale, malware-free intelligence collection campaign in which APT28 compromised tens of thousands of small office and home office (SOHO) routers worldwide, altered their DNS configurations to redirect authentication traffic through attacker-controlled adversary-in-the-middle (AitM) proxy nodes, and harvested Microsoft 365 credentials, OAuth session tokens, and email content from targeted organizations without deploying any endpoint malware.

**When:** Router exploitation and DNS redirection activity commenced as early as mid-2024. The campaign escalated significantly on 6 August 2025, the day following the UK National Cyber Security Centre's (NCSC) publication of a related advisory on Forest Blizzard credential-stealing tooling. Activity reached peak scale in December 2025, with approximately 18,000 victim IP addresses confirmed within a single 30-day observation window. On 7 April 2026, the U.S. Department of Justice and FBI publicly disclosed and disrupted the operation through a court-authorized action designated Operation Masquerade, with coordinated disclosures from 16 allied nations released simultaneously.

**Where:** Victim organizations spanned more than 120 countries. Confirmed impacted sectors include government ministries of foreign affairs, national law enforcement agencies, military organizations, defense contractors, and critical infrastructure operators. At peak scale, approximately 290,000 distinct IP addresses from consumer and enterprise networks worldwide were communicating with attacker-controlled DNS infrastructure.

**Why:** The operation served Russian state intelligence objectives by enabling mass, passive interception of authentication credentials and email communications from high-priority government, military, and strategic targets. The campaign architecture reflects a deliberate avoidance of endpoint malware deployment, substantially reducing the likelihood of detection by host-based security controls and incident response processes. The broad initial router exploitation phase provided a wide access pool from which the actor selectively activated AitM interception against intelligence-priority victims through automated traffic filtering, making the operation both scalable and operationally discreet.

---

## Background

The campaign, designated FrostArmada by Lumen Technologies' Black Lotus Labs, represents a technically significant evolution in APT28's operational tradecraft. Unlike prior Forest Blizzard operations that relied on spear-phishing, credential stuffing, or bespoke malware delivery, FrostArmada operated entirely within the network infrastructure layer. By modifying router DNS settings, the actor positioned itself upstream of all endpoint security controls on every network served by a compromised router, rendering conventional antivirus, endpoint detection and response (EDR), and network-based signature detection functionally blind to the interception activity.

Two government disclosures are directly relevant to this incident.

FBI/IC3 Public Service Announcement (7 April 2026): Initial public disclosure of GRU router exploitation activity.
https://www.ic3.gov/PSA/2026/PSA260407

DOJ Press Release (7 April 2026): Announcement of the court-authorized Operation Masquerade disruption action.
https://www.justice.gov/opa/pr/justice-department-conducts-court-authorized-disruption-dns-hijacking-network-controlled

At the campaign's December 2025 peak, Lumen's Black Lotus Labs observed approximately 40,000 low-confidence and 18,000 moderate-confidence victim IP addresses across more than 120 countries communicating with attacker infrastructure. Over 200 organizations and more than 5,000 consumer devices were confirmed impacted across government, military, law enforcement, and critical infrastructure sectors. The Operation Masquerade takedown was notable for its multinational scope: the U.S. court-authorized action remediated compromised routers by resetting DNS configurations and revoking attacker access, while MIT Lincoln Laboratory independently verified that the remediation commands did not impair normal router functionality.

---

## Technical Details

The attack chain comprised three sequential phases, none of which required the deployment of malware on any victim endpoint. The entire operation was conducted through DNS infrastructure manipulation and transparent proxy services, a tradecraft approach that Lumen researcher Ryan English characterized as one designed to evade detection precisely because it does not produce the malware artifacts that security teams are trained to identify.

**Phase 1: Router Exploitation.** An actor sub-cluster designated the "Expansion Team" conducted internet-wide scanning of TCP port 80 to identify exposed SOHO routers. Against TP-Link devices, the actor exploited CVE-2023-50224, an authentication bypass vulnerability in the httpd service of the TP-Link WR841N that permits unauthenticated credential extraction from the `/tmp/dropbear/dropbearpwd` path via crafted HTTP GET requests. A subsequent crafted request granted full administrative access to the device. The NCSC confirmed that 23 distinct TP-Link router models were targeted, along with MikroTik routers, end-of-life Fortinet firewalls, and Nethesis firewalls.

**Phase 2: DNS Poisoning.** Upon gaining administrative access to a router, the actor modified the device's DHCP DNS settings, substituting an attacker-controlled virtual private server (VPS) IP address as the primary DNS server while preserving the legitimate ISP DNS server as the secondary entry. All downstream devices on the affected network inherited the poisoned DNS configuration automatically via DHCP, requiring no user interaction whatsoever. The malicious VPS servers ran dnsmasq version 2.85 on UDP port 53 as recursive resolvers. For non-targeted domains, queries resolved to legitimate addresses, maintaining the appearance of normal network function for users on the compromised network. For targeted domains, specifically Microsoft Outlook authentication endpoints including `autodiscover-s.outlook.com`, `outlook.office365.com`, and `imap-mail.outlook.com`, the resolver returned the IP address of the attacker's AitM proxy node instead of the legitimate Microsoft service address.

**Phase 3: Credential and Token Harvesting.** When users on compromised networks authenticated to email or cloud services, their traffic was silently redirected to an AitM proxy node. The proxy presented its own TLS certificate, the sole visible indicator of interception being a browser certificate warning. The proxy then forwarded the connection transparently to the legitimate Microsoft service. Users who dismissed the certificate warning completed their full authentication flow, including any multi-factor authentication (MFA) challenges, while the proxy captured plaintext passwords, OAuth tokens, session cookies, and email content at the interception midpoint. This mechanism effectively circumvented multi-factor authentication by harvesting valid post-authentication session tokens before they could expire.

Two primary AitM nodes were identified during the investigation: **64.120.31[.]96**, active from 19 May 2025 and initially observed targeting Afghan government entities, and **79.141.160[.]78**, active from 6 August 2025, with global reach and DNS-over-TLS (DoT) capability. The campaign operated through two distinct infrastructure clusters. Cluster 1 handled global-scale DNS hijacking across approximately 130 or more malicious IP addresses. Cluster 2 focused on Ukraine-specific operations through compromised MikroTik routers across approximately 33 IP addresses, reflecting prioritization of military intelligence collection in the active conflict theater.

---

## Fix Actions

| Priority | Action | Detail | Owner |
|----------|--------|--------|-------|
| 1 | Replace or isolate all end-of-life SOHO routers in remote and hybrid work environments immediately | The device models exploited in this campaign will not receive vendor security patches. Organizations must establish enforceable policies preventing access to corporate resources from networks equipped with unmanaged or end-of-support edge devices. | IT / Network Operations |
| 2 | Enforce certificate pinning on all corporate-managed endpoints via mobile device management (MDM) | Certificate pinning generates a hard failure rather than a dismissible warning when AitM proxies attempt TLS interception. This control directly neutralizes the interception mechanism used throughout this campaign and should be treated as a required baseline for remote workforce devices. | Endpoint / MDM Team |
| 3 | Deploy Zero Trust DNS (ZTDNS) on all managed Windows endpoints and monitor router DNS configurations for unauthorized modification | A router configured with an unrecognized primary DNS server and the original ISP server as secondary is a reliable indicator of compromise consistent with this attack pattern. Automated monitoring should alert on any such configuration change across managed and remote-worker devices. | SOC / Network Team |
| 4 | Harden Conditional Access policies to block authentication from known VPS IP ranges and detect anomalous session token behavior | Microsoft Entra ID Protection should be configured to flag risk-level-100 sign-in events that nonetheless succeed (ErrorCode 0 or 50140) as indicators of AitM interception. The recommended Microsoft advanced hunting query is: `AADSignInEventsBeta | where RiskLevelAggregated == 100 and (ErrorCode == 0 or ErrorCode == 50140)` | Identity / Security Team |
| 5 | Review DNS logs for targeted Microsoft authentication domains resolving to non-Microsoft IP addresses | Any resolution of `autodiscover-s.outlook.com`, `imap-mail.outlook.com`, `outlook.live.com`, `outlook.office.com`, or `outlook.office365.com` to addresses outside Microsoft's published IP ranges should be investigated as a potential indicator of active DNS hijacking on the querying network. | SOC / Detection Team |
| 6 | Deliver user awareness training on TLS certificate warnings as potential indicators of active network-level interception | Users must be instructed to treat unexpected certificate warnings on familiar services as security events to be reported immediately, rather than routine prompts to be dismissed. This behavioral control provides a last-resort detection signal against AitM operations that bypass all technical controls. | Security Awareness Team |

---

## Indicators of Compromise

### MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique | Confidence |
|--------|-------------|-----------|------------|
| Initial Access | T1190 | Exploit Public-Facing Application: CVE-2023-50224 and related SOHO router vulnerabilities exploited via internet-facing administrative interfaces. | HIGH |
| Credential Access | T1557 | Adversary-in-the-Middle: AitM proxy nodes intercepted TLS sessions to harvest credentials, OAuth tokens, and session cookies from authenticating users. | HIGH |
| Resource Development | T1583.002 | Acquire Infrastructure: DNS Server: Malicious dnsmasq-2.85 recursive resolvers operated on attacker-controlled VPS nodes on UDP port 53. | HIGH |
| Resource Development | T1583.003 | Acquire Infrastructure: Virtual Private Server: VPS nodes configured as DNS resolvers and AitM proxy endpoints for credential interception. | HIGH |
| Resource Development | T1584.008 | Compromise Infrastructure: Network Devices: TP-Link, MikroTik, Fortinet, and Nethesis SOHO devices compromised via vulnerability exploitation and default credential abuse. | HIGH |
| Resource Development | T1588.006 | Obtain Capabilities: Vulnerabilities: Actor leveraged CVE-2023-50224 and additional publicly disclosed router firmware vulnerabilities. | MEDIUM |
| Credential Access | T1586 | Compromise Accounts: Valid OAuth tokens and authentication credentials harvested via AitM interception and used for subsequent intelligence collection. | HIGH |
| Defense Evasion | T1562 | Impair Defenses: The operation was conducted entirely without endpoint malware deployment, rendering host-based security controls ineffective against the interception chain. | HIGH |

### APT Infrastructure

| Type | Indicator | Description | Confidence |
|------|-----------|-------------|------------|
| IP | 64.120.31[.]96 | Primary AitM proxy node; active from 19 May 2025; initially observed targeting Afghan government entities. | HIGH |
| IP | 79.141.160[.]78 | Primary AitM proxy node; active from 6 August 2025; global targeting scope with DNS-over-TLS capability. | HIGH |
| Port | TCP 56777 | SSH service signature characteristic of Cluster 1 VPS nodes alongside dnsmasq-2.85 on UDP 53. | HIGH |
| Port | TCP 35681 | SSH service signature characteristic of Cluster 2 Ukraine-focused VPS nodes. | HIGH |
| IP Range | 5.226.137.x, 37.221.64.x, 77.83.197.x | Cluster 1 DNS resolver address ranges per Lumen Black Lotus Labs attribution. | MEDIUM |
| IP Range | 79.141.160.x through 79.141.173.x, 185.117.88.x, 185.117.89.x, 185.237.166.x | Additional Cluster 1 and Cluster 2 infrastructure ranges. | MEDIUM |
| Domain | autodiscover-s.outlook[.]com | Legitimate Microsoft Outlook autodiscovery endpoint; targeted for DNS spoofing to redirect authentication traffic. | HIGH |
| Domain | imap-mail.outlook[.]com | Legitimate Microsoft IMAP endpoint; targeted for DNS spoofing. | HIGH |
| Domain | outlook.live[.]com, outlook.office[.]com, outlook.office365[.]com | Additional Microsoft 365 authentication endpoints targeted for DNS redirection. | HIGH |
| CVE | CVE-2023-50224 | Authentication bypass in the TP-Link WR841N httpd service; exploited for unauthenticated administrative access to router configurations. | HIGH |
| Tool | dnsmasq 2.85 | Legitimate open-source DNS forwarder repurposed as an attacker-controlled recursive resolver on UDP port 53; detect via context (unexpected presence on commercial VPS infrastructure) rather than the tool name alone. | MEDIUM |

The complete IOC lists for Cluster 1 (130 or more IP addresses) and Cluster 2 (33 IP addresses) are maintained by Lumen's Black Lotus Labs at the following repository:
https://github.com/blacklotuslabs/IOCs/blob/main/FrostArmada_IOCs.txt

---

## Summary

Between mid-2024 and April 2026, APT28 conducted a sustained, malware-free intelligence collection campaign that compromised more than 18,000 SOHO routers across 120 or more countries by exploiting unpatched firmware vulnerabilities, most notably CVE-2023-50224 in TP-Link devices. By altering router DNS configurations to redirect Microsoft 365 authentication traffic through attacker-controlled AitM proxy nodes, the actor harvested credentials, OAuth session tokens, and email content from more than 200 organizations worldwide without triggering any host-based security controls. On 7 April 2026, Operation Masquerade remediated compromised U.S. routers and publicly exposed the campaign's full scope through a coordinated 16-nation disclosure. The simultaneous deployment of the PRISMEX malware suite against Ukraine and NATO logistics partners during the same operational period confirms that FrostArmada represents one component of a broader, multi-vector Russian offensive cyber posture combining passive infrastructure-level collection with active targeted intrusion and, in at least one confirmed instance, destructive wiper capability.

---

## References

https://www.ic3.gov/PSA/2026/PSA260407

https://www.justice.gov/opa/pr/justice-department-conducts-court-authorized-disruption-dns-hijacking-network-controlled

https://www.fbi.gov/contact-us/field-offices/philadelphia/news/justice-department-conducts-court-authorized-disruption-of-dns-hijacking-network-controlled-by-a-russian-military-intelligence-unit

https://media.defense.gov/2026/Apr/07/2003907743/-1/-1/0/I-260407-PSA.PDF

https://www.ncsc.gov.uk/news/apt28-exploit-routers-to-enable-dns-hijacking-operations

https://www.microsoft.com/en-us/security/blog/2026/04/07/soho-router-compromise-leads-to-dns-hijacking-and-adversary-in-the-middle-attacks/

https://www.lumen.com/blog-and-news/en-us/frostarmada-forest-blizzard-dns-hijacking

https://github.com/blacklotuslabs/IOCs/blob/main/FrostArmada_IOCs.txt

https://www.bleepingcomputer.com/news/security/authorities-disrupt-dns-hijacks-used-to-steal-microsoft-365-logins/

https://krebsonsecurity.com/2026/04/russia-hacked-routers-to-steal-microsoft-office-tokens/

https://thehackernews.com/2026/04/russian-state-linked-apt28-exploits.html

https://thehackernews.com/2026/04/apt28-deploys-prismex-malware-in.html

https://www.trellix.com/blogs/research/apt28-stealthy-campaign-leveraging-cve-2026-21509-cloud-c2/

https://securityaffairs.com/190510/apt/russia-linked-apt28-uses-prismex-to-infiltrate-ukraine-and-allied-infrastructure-with-advanced-tactics.html

https://www.darkreading.com/threat-intelligence/russia-forest-blizzard-logins-soho-routers

https://www.infosecurity-magazine.com/news/us-thwarts-dns-hijacking-network/

https://industrialcyber.co/critical-infrastructure/uk-ncsc-says-apt28-exploits-routers-for-dns-hijacking-enabling-large-scale-traffic-interception/

https://attack.mitre.org/groups/G0007/

---
Author: Josephino Cambosa

Date: 2026-04-12
