---
title: Iranian-linked Stryker Cyber Attack
date: 2026-04-07
categories: [APT, Hacktivism, Malware]
tags: [cyberattack, APK, DDoS attacks, Hacktivism, Iran, Phishing, Wiper]
image:
  path: /assets/img/blog/2026-04-07/thumbnail-stryker-attack.png
---
## Overview

![Void Manticore online personas including Handala, Karmabelow80, and Homeland Justice](/assets/img/blog/2026-04-07/void-manticore-personas.png)

**Who:** The Handala Hack Team (also tracked as Void Manticore, Red Sandstorm, Banished Kitten, Storm-0842) is reported as a pro-Palestinian hacktivist identity as of 2023. Handala is assessed as a prominent online persona of Void Manticore, a threat actor with known affiliation to the Ministry of Intelligence of the Islamic Republic of Iran. This threat actor has claimed responsibility for the following incidents:
- [Compromising an Israeli energy exploration company](https://industrialcyber.co/industrial-cyber-attacks/us-israeli-campaign-triggers-iranian-counteroffensive-targeting-gulf-energy-critical-infrastructure/)
- [Compromising Jordanian fuel systems](https://socradar.io/blog/dark-web-profile-handala-hack/)
- [Targeting Israeli civilian healthcare](https://ransomwareattacks.halcyon.ai/attacks/ransomware-attack-on-sheba-medical-center-by-handala-exposes-sensitive-data)

**What:** The widespread execution of a malicious wiping campaign after Stryker’s internal Microsoft Intune mobile device management console was compromised, resulting in the factory reset of corporate laptops, tablets, and mobile devices.

**When:** Attack began on March 11, 2026 after midnight (EST).

**Where:** Stryker Corporation employees were impacted globally.

**Why:** Following the U.S.-Israeli strikes on February 28, 2026, Iran responded with a coordinated multidomain counteroffensive, shifting the initial engagement into a broad transregional war. As a prominent Iranian-aligned nation-state actor, Handala primarily targets organizations for the purposes of clandestine intelligence gathering alongside efforts to disrupt critical infrastructure and services. The Handala Stryker Corporation attack is reported to be the most impactful Iran-attributed attack against a Western company during this ongoing conflict.

---
## Background

Over 200,000 corporate laptops, tablets, and mobile devices across 79 countries were remotely wiped and had the Handala logo set as their login screens.

There are two relevant 8-K filings to the SEC that capture this incident.

8-K (March 11, 2026): Initial disclosure
https://www.sec.gov/Archives/edgar/data/310764/000119312526102460/d76279d8k.htm

8-K (March 12, 2026): Operational update, confirms order processing, manufacturing, and shipping disruption
https://www.sec.gov/Archives/edgar/data/0000310764/000119312526104431/d101097d8k.htm

## Technical Details
The attacker obtained a compromised Microsoft 365 Global Admin account. With this access, the attacker executed the wiping campaign through admin control-plane abuse by means of the [`wipe` remote device action](https://learn.microsoft.com/en-us/intune/device-management/actions/wipe?pivots=windows) from the Microsoft Intune admin center. Microsoft Learn outlines the following steps:
1. In the Microsoft Intune admin center, select Devices > All devices.
2. From the devices list, select a device.
3. At the top of the device overview pane, find the row of remote action icons. Select Wipe.

## Fix Actions

| Priority | Action | Detail | Owner |
|----------|--------|--------|-------|
| 1 | Enforce multi-admin approvals for bulk remote wipes and MFA (FIDO2 hardware keys) on all MDM admin accounts | Require step-up authentication for any bulk device retire, reset, or wipe commands in Microsoft Intune. Prevents single-point administrative misuse from causing enterprise-wide device loss. | Identity / Platform Team |
| 2 | Prioritize identity and MDM privilege hardening | Enable Conditional Access and restrict privileged admin sign-ins by location and device posture. Limits attacker ability to abuse identity-plane controls for destructive actions. | Identity / IT |
| 3 | Deploy detection logic for mass remote-wipe activity across managed devices | Correlate Intune/MDM admin commands with device check-ins, unusual timing, and admin session geolocations. Alert on bulk retire/reset activity, sudden privilege escalation, or step-ups from new IP ranges. | SOC / Detection Team |

## Indicators of Compromise

### MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique | Confidence |
|--------|--------------|-----------|------------|
| Initial Access | T1133 | External Remote Services - vendor reporting includes compromised remote access; not Stryker-confirmed. | LOW-MED |
| Credential Access | T1003.001 | OS Credential Dumping: LSASS Memory - reported in broader actor tradecraft; not Stryker-confirmed. | LOW-MED |
| Discovery | T1087.002 | Account Discovery: Domain Account - relevant if identity/MDM abuse occurred. | LOW-MED |
| Lateral Movement | T1021.001 | Remote Services: RDP - seen in vendor reporting for actor operations; not Stryker-validated. | LOW-MED |
| Defense Evasion | T1572 | Protocol Tunnelling - NetBird usage in vendor reporting supports tunnelling capabilities. | MEDIUM |
| Persistence / Execution | T1053.005 | Scheduled Task - vendor reports include scheduled tasks/GPO deployment in actor playbook. | LOW-MED |
| Impact | T1485 | Data Destruction - strong alignment with observed wipe-like effects at Stryker. | HIGH |
| Impact | T1561.001/T1561.002 | Disk Wipe / Disk Structure Wipe - relevant to wiper tradecraft reported for Handala. | MEDIUM |

### APT Infrastructure

| Type | Indicator | Description | Confidence |
|------|-----------|-------------|------------|
| Domain | handala-hack[.]to | Actor claim/propaganda hosting; non-resolving at collection time. | MEDIUM |
| Domain | handala[.]to | Alternate actor domain; non-resolving at collection time. | MEDIUM |
| URL | t[.]me/HANDALA_HPR2 | Actor Telegram channel used for claims and messaging. | MEDIUM |
| IP | 107[.]189[.]19[.]52 | Vendor-associated infrastructure; no public scan confirmation. | MEDIUM |
| IP | 31[.]57[.]35[.]223 | Vendor-cited IP with exposed RPC/SMB services; contextual candidate. | MEDIUM |
| IP | 82[.]25[.]35[.]25 | Vendor-cited IP with exposed RPC/SMB services; contextual candidate. | MEDIUM |
| IP | 146[.]185[.]219[.]235 | Vendor-cited VPN/exit node candidate; likely shared infrastructure. | LOW |
| SHA256 | 5986ab04dd6b3d259935249741d3eff2 | Reported Handala wiper sample. | MEDIUM |
| MD5 | 3cb9dea916432ffb8784ac36d1f2d3cd | Reported PowerShell wiper sample. | MEDIUM |
| SHA256 | 3236facc7a30df4ba4e57fddfba41ec5 | VeraCrypt installer - dual-use tooling; context required before blocking. | LOW |
| SHA256 | 3dfb151d082df7937b01e2bb6030fe4a | NetBird installer - dual-use tunnelling tool; detect via context, not name. | LOW |

## Summary
On March 11, 2026, Handala leveraged compromised Intune credentials to initiate a global device-wipe campaign against Stryker Corporation. The attack bypassed traditional defenses to factory reset over 80,000 endpoints without the use of malware or ransomware, causing a multi-day suspension of global manufacturing and electronic ordering before the incident was contained.

## References
https://www.stryker.com/us/en/about/news/2026/a-message-to-our-customers-03-2026.html

https://unit42.paloaltonetworks.com/iranian-cyberattacks-2026/

https://unit42.paloaltonetworks.com/threat-bulletin/march-2026/

https://www.protoslabs.io/resources/timeline-reconstruction-stryker-handala-threat-group-wiper-attack

https://www.seqrite.com/blog/iran-us-israel-cyberwar-2026-analysis/

https://research.checkpoint.com/2026/handala-hack-unveiling-groups-modus-operandi/

https://arcticwolf.com/resources/blog/stryker-systems-disrupted-cyber-attack-handala-group-claims-responsibility/

https://hackread.com/iran-handala-hackers-verifone-stryker-hacks/

https://www.bleepingcomputer.com/news/security/stryker-attack-wiped-tens-of-thousands-of-devices-no-malware-needed/

---
Author: Josephino Cambosa

Date: 2026-04-07
