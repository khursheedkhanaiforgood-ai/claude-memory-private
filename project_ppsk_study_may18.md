---
name: PPSK Study + EP1 Deployment Guide — May 18–19 2026
description: PPSK guide Socratic May 18. ep1-deployment-guide.html v2.4 on main (commit ccdf482). ALL corpus docs read May 19: User Guide AI Expert pp.37-46, Switch Guide all 130pp, Licensing Guide all 39pp, Patches 1-4. RAG corpus v2.4.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## What Was Done
- Read entire 95-page PPSK Ultimate Guide v6 by Mike Rieben (SA) cover to cover — May 18 2026
- Built 3-hour deploy-ready study plan with 5 blocks + drill questions
- Context: Karl's lab had bare-bones PPSK deployed Friday May 15. Monday May 18 = expand/harden.

## Key Facts to Include in EOD HTML

### Architecture
- PPSK delivery: Client PSK auth → RadSec Proxy AP → Cloud (port 2083 TCP/UDP) → PMK back to proxy → distributed to neighbor APs → 4-Way Handshake
- Two storage options: Cloud (up to 100K users/UG, 32 UGs/SSID) vs Local (1K users/UG, 10K total, up to 1000 keys/group)
- RadSec proxy: 2 APs auto-elected per management subnet. Not configurable. Visible in Manage > Devices.

### Hard Constraints (deploy-critical)
- Cannot mix Cloud + Local User Groups in same SSID
- MAC Binding = Local DB only
- AP-Based PCG wired port assignment = Local DB + AP150W/AP302W only
- PPSK Classification = Local DB + map hierarchy in XIQ required
- SMS not delivered with PPSK Classification feature
- Only one SSID per PCG type per Network Policy (one AP-Based + one Key-Based max)
- PPSK does NOT support WPA3/SAE (as of v6 guide)

### Most Commonly Skipped Steps
1. Key-Based PCG: must add users to PCG list (Configure > Users > Private Client Groups) or segmentation silently fails
2. User Group created OUTSIDE SSID → emails NOT delivered even if option selected
3. After any Local PPSK change → push Delta config update to APs
4. After Classification change → push Complete Configuration Update (reboot required)

### Guest Management Role
- Role: Guest Management (Configure > Accounts > Account Manager)
- Credential Distribution Groups: controls which User Groups, daily limit, email approval
- Guest manager sees ONLY Users page. Can only manage keys they created.
- 2FA via Google Auth recommended for HIPAA/PCI compliance

### PCG Decision Matrix
- AP-Based: room isolation, GRE tunnel back to anchor AP, 4 individual + 1 shared key per room, First AP Problem (64 GRE tunnel limit), Local DB preferred
- Key-Based: same-key devices communicate, different-key devices isolated via hidden ACLs, no GRE, Cloud or Local, popular choice

### Guest Onboarding
- Dual SSID: PPSK-secured + Open/CWP. Guest switches SSIDs after getting PPSK.
- Single SSID: Open + CWP + User Auth. Password Type = RADIUS (not PPSK). Auth = MS-CHAPv2.
- Employee Sponsorship: sponsor gets email → clicks Approve → guest gets key
- CWP HTML Hack: hardcode sponsor email alias, hide field from guest (index.html edit)

### CLI Troubleshooting
- `#show idm` — check port 2083, RadSec cert Valid, RUN state Connected
- `#_show radsec elct-pool` — shows elected proxy APs (up to 2 per subnet)
- `#show roaming cache mac XXXX:XXXX:XXXX` — VLAN ID must = 0
- RadSec cert invalid → check NTP first; then clear + force regen
- Locked Devices: 10 attempts / 7 min → 30 min block. Manage > Tools > Utilities > Locked Devices

## Study Plan File
Stored in conversation context. 5 blocks × 3 hours:
- Block 1 (0:00-0:45): Foundation + Decision Architecture
- Block 2 (0:45-1:30): Classification + MAC Binding + Guest Roles
- Block 3 (1:30-2:15): Private Client Groups (AP-Based vs Key-Based)
- Block 4 (2:15-2:45): Guest Onboarding + CWP
- Block 5 (2:45-3:00): CLI Troubleshooting Sprint

## Socratic Session Progress — May 18 2026

### Completed ✅
- SSID structure: two SSIDs needed (Employee + BYOD separate) — different schedules require separate SSIDs for broadcast control
- 1 device per key: checkbox "Set max clients per private PSK" = 1 at SSID level. User Group level overrides. Two SSIDs solves cleanly.
- Data rates: 18Mbps = Basic. Above 18 = Supported/Optional. Below 18 = N/A. Found in Additional Settings > Optional Settings > Radio and Rates > 5GHz
- Interstation communication: Additional Settings > Optional Settings > Traffic Filters > "Enable Inter-station Traffic" — UNCHECKED = blocked
- Multicast-to-unicast: Convert IP Multicast to Unicast = Always
- Multicast Drop: Enable Multicast Drop. Exceptions kept: DHCPv4, DHCPv6, ARP, IGMP-query. DROPPED (unchecked): IPv6-Discovery, MDNS
- Guest CWP: Open + UPA (Use Policy Acceptance) = correct for T&C acceptance
- Multicast fundamentals taught: Convert = deliver better (unicast per client). Drop = discard entirely.

### Completed ✅ (continued)
- Firewall logic: DENY RFC 1918 (10/8, 172.16/12, 192.168/16) top-down, PERMIT ANY/ANY last = internet only
- Guest VLAN: task silent on Guest VLAN number. Professional decision = VLAN 15 (keep VLAN 1 clean for AP mgmt). Documented reasoning.
- VLAN map final: VLAN 1=AP Mgmt, VLAN 5=BYOD, VLAN 10=Employee, VLAN 15=Guest
- Firewall applies to both BYOD and Guest (same rule set)
- Multicast: ARP/DHCP are broadcast not multicast — don't conflate. They appear in MC drop as protective exceptions only.

### Brain Map — How Learning Progressed
- Started with shared PSK vs PPSK distinction (key compromise containment)
- Segmentation: VLAN-based (User Profile) vs client isolation (PCG) — needed teaching before recognition
- Found interstation setting only after being guided to Additional Settings > Optional Settings
- Confused 192.168.0.0 as internet address (corrected: RFC 1918 = private)
- Correctly self-identified Guest-on-VLAN-1 as security risk unprompted
- Firewall rule order (first match wins) understood once explained
- Multicast: needed 2-min foundation before could apply drop vs convert distinction
- EXOS question: raised independently — good infrastructure thinking

### Identified Knowledge Gaps
1. Where interstation setting lives in XIQ (needed guidance to Additional Settings)
2. RFC 1918 address ranges not memorized
3. Firewall rule direction/order logic (top-down first match)
4. Multicast fundamentals (convert vs drop)
5. Roaming cache — logic understood, XIQ location unknown
6. NTP in XIQ — location unknown
7. SNMP on AP — never deployed manually, only saw defaults
8. EXOS trunk/VLAN requirements for AP connectivity

### Still To Cover
- Employee: SSID availability schedule at SSID level (broadcast window)
- BYOD: same + interstation + firewall confirmed in XIQ
- Guest: interstation + firewall + VLAN 15 confirmed in XIQ
- DNS/NTP/ARP suppression/Roaming cache (segment 2)
- SNMP configuration on AP (segment 3)
- EXOS switch question: what VLANs/trunk config needed on AP port

## EOD HTML Sections Needed
- PPSK Architecture diagram (text-based)
- Feature decision matrix (Cloud vs Local, PCG type, Guest flow)
- Drill questions + answers (all 5 blocks)
- CLI command reference table
- What was deployed today on Karl's lab

**Why:** Khursheed needs PPSK deployable from memory for client work. This session is the knowledge foundation.

## May 19 — Architecture Analysis Session

### Deliverables Completed
- ep1-deployment-guide.html bumped to v2.7 — committed + pushed, commit fe227f6 on main
- xiq-ep1-arch-debate.html created — verbatim dialogue log of the XIQ vs EP1 architectural debate, 10 sections, annotation boxes for engineering/R&D team
- session_summary_20260519.html updated with architecture section and team call-to-action

### Three-Flow Analysis
1. Policy Push — how configuration intent travels from EP1 to APs
2. Telemetry/Observability — how AP telemetry flows back to the cloud analytics stack
3. AP Optimization — how AI Core drives RRM and tuning decisions

### Primary Source
EP1 Cloud Architecture & Security white paper (flipbook-with-notes.pdf, 13pp, ©2025 EN) — fully page-referenced throughout the debate log.

### Citation Discipline
- All confirmed facts carry exact page citations: p.2, p.3, p.6, p.7, p.8, p.9, p.10, p.12, p.13
- All hypothesized items are explicitly labeled HYPOTHESIS — includes: DB tech names, multi-agent optimization workflow, Kafka event bus, Iceberg lakehouse, XIQ internal DB architecture

### Open Questions
- 7 items pending R&D verification — catalogued in xiq-ep1-arch-debate.html Section 9

### Team Engagement
- Extended engineering team (30+ yrs industry, 15+ yrs Extreme per person) asked to annotate Section 10 of the debate log

### Session Status
- Session paused — user will review later in day
- PPSK EOD HTML for the May 18 session (5320 landing page entry) still PENDING — was not completed this session
