---
name: Extreme Control (NAC) Documentation Index
description: Categorized index of Extreme Control / ExtremeControl NAC docs across KB + internal SharePoint, plus architecture facts (UZTNA relationship, Site Engine access path) and local folder location. Built Sept 8 2026 for the Extreme Control learning session.
type: reference
originSessionId: 7403a6e9-ad49-464b-9285-c295b912c840
---
# Extreme Control (NAC) — Documentation Index

Built for a session on learning ExtremeControl: how it works, how to deploy/configure, how it complements/competes with other NAC solutions (Cisco ISE, Aruba ClearPass), and how it relates to Universal ZTNA / Extreme Platform ONE.

## Local folder (already exists, OneDrive-synced)
`/Users/khukhan/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Documents/Extreme_Product_Reference/Extreme_Control_NAC/`
(symlinked from `~/OneDrive - Extreme Networks, Inc/Documents/Extreme_Product_Reference/Extreme_Control_NAC/`)

As of Sept 8 2026, contained:
- `architecture/` → ExtremeCloud IQ Site Engine Security Integration Guide.pdf
- `configuration/` → ExtremeControl NAC User Guide 26.02.10.pdf, ExtremeControl Guest and IoT Manager Configuration Guide 22.09.10.pdf
- `deployment/` → Upgrading to ExtremeControl 23.07.10.pdf (upgrade guide only — no full "Deploy with XIQ" guide yet)

New subfolders (`demos_and_workshops/`, `competitive_positioning/`, `datasheets/`) were created and populated via a bulk SharePoint→OneDrive copy — see the "Bulk copy" section below for what landed and what didn't.

## Architecture / How It Works
- ExtremeControl = centralized NAC, delivered as an **application within ExtremeCloud IQ – Site Engine** (not a standalone product, not cloud-native).
- Mechanism: builds contextual identity per endpoint (user, time, location, vulnerability, access type) → applies role-based access consistently regardless of connection point → enforces via downloadable/dynamic ACLs, VLANs, and Fabric Connect Service Identifiers (L2VSN/L3VSN I-SIDs) → integrates with NGFW/SIEM/CMDB/EMM-MDM for orchestrated isolation/remediation.
- **Access path confirmed via KB**: managed from within ExtremeCloud IQ Site Engine — click "Control" in the top menu bar → opens the Control tab (sub-tabs: Dashboard, Policy, Access Control, End-Systems, Reports). Inside Access Control, three left-panel nav trees: ExtremeControl Configuration, ExtremeControl Group Editor, All ExtremeControl Engines. **So yes: for on-prem NAC, you get to Control via Site Engine — confirmed, not inferred.**
- SharePoint architecture docs: `NAC in Campus Fabric Edge.pptx` (Fabric Connect integration), `ExtremeCloud IQ Site Engine Security Integration Guide.pdf`, `Radius vsa for Fabric Attach.pptx`, `Fabric & Switch Engine radsec configuration.pptx`.

## ExtremeControl vs. Universal ZTNA (UZTNA) / Extreme Platform ONE — CORRECTED
A claim surfaced from an external source during this session ("ExtremeControl is included in Extreme Platform ONE") is **not confirmed by KB — flagged as inaccurate as stated**:
- ExtremeControl = on-prem NAC, delivered as a Site Engine app. It is a **separate product line**, not a Platform ONE component.
- UZTNA = broader zero-trust access (network/app/device, any location, cloud/private-cloud/on-prem deployable) that ships as **Extreme Platform ONE Security, an additional paid subscription** — NOT bundled into base Platform ONE licensing.
- KB explicitly does NOT state ExtremeControl is the on-prem equivalent of UZTNA, nor that it's included in standard EP1 licensing. The two are **related but distinct**: ExtremeControl = local/on-prem enforcement for compliance-driven environments; UZTNA = cloud-managed zero trust for users/devices regardless of location. Positioning language calls them complementary, not interchangeable.
- **Practical implication**: if the goal is "deploy EP1 with UZTNA" per this session's stated intent, that's a distinct product track from ExtremeControl/Site Engine — don't assume standing up one gives you the other. Verify current licensing/bundling directly with Product Marketing before quoting this to a customer; KB's own sources hedge on the exact licensing boundary.

## Deployment Guides
- `ExtremeControl HOW-TO Guide - Deploy with XIQ Native.pdf` / `Deploy with XIQ.pdf` — official step-by-step (site: se-access-security, and kcs/External).
- `XIQSE_21.09.10_Control_Analytics_Virtual_Engine_Installation_Guide.pdf` — virtual engine install (remote-demo-lab CSE Sandboxes).
- `Extreme Control Integration.pdf/docx` (2017, SE Training) — legacy but still-cited original NAC install/config walkthrough.

## Configuration Guides
- `ExtremeControl NAC User Guide 26.02.10.pdf` — current authoritative config reference (already local).
- `ExtremeControl Guest and IoT Manager Configuration Guide 22.09.10.pdf` (already local).
- `ExtremeCloudIQ-SiteEngine_ExtremeControl_Cisco-Integration_Guide.pdf` — ExtremeControl driving Cisco switches.
- `ExtremeCloudIQ-SiteEngine_ExtremeControl_VOSS_ACL_Guide.pdf` — VOSS/Fabric Engine downloadable ACLs (directly relevant to the KhKLab-SW-01/SW2 VOSS lab from other sessions).
- `ExtremeControl and Meraki switch.pdf`, `ExtremeControl with WiNG 5.8.pdf` — third-party/legacy integration.
- `ExtremeCloud Appliance and ExtremeControl Captive Portal Configuration Lab Guide` — GTAC captive-portal lab (site: GTAC/Shared Documents/XMC).
- KB-confirmed RADIUS/AAA best practices: Proxy RADIUS via the Access Control Engine; primary+secondary engine redundancy; `RADIUS Accounting: Enabled`; disable local MAC auth for external captive portal; `Filter-Id=%POLICY_NAME%` RADIUS attribute schema for switch integration.

## Demos / Training / Workshops — user flagged as highest priority for hands-on learning
- **"Extreme Control - Fabric to the Edge"** — CSE remote-demo-lab live sandbox + full Lab Guide, deploying ExtremeControl against a Fabric (SPB) topology. **Closest thing to a step-by-step deployment walkthrough found** — directly usable alongside the KhKLab-SW-01/SW2 VOSS fabric already built in other sessions.
- **"Extreme Control Workshop"** folder (Texas Tech University, USSLED-Central site, "Uploads from Jeff") — **real-world precedent**: documents an actual customer ExtremeControl deployment engagement, including a demo transcript covering ExtremeControl coexisting with an existing Aruba wireless deployment.
- `ExtremeControl_Workshop_Apr2025/Aug2024/May2024` decks + `Student1–6_NACFabricWorkshop_v2.1.pdf` hands-on student labs (se-americas site).
- `Extreme Control Configuration.pptx` + `Extreme Control Troubleshooting.pptx` — Connect 2025 hands-on course decks.
- `AccessControlDemoLabOverview.pdf` — remote demo lab walkthrough.

## Data Sheets / Overview Decks
- `extremecontrol-data-sheet.pdf` (2018 baseline) / `2020-02 ExtremeControl R8.4 Datasheet.pdf`.
- `ExtremeCloud IQ Site Engine Data Sheet - 26Jun2026.pdf` — current, positions ExtremeControl as a licensed app within XIQ-SE.

## Competitive Positioning
- **vs Cisco ISE — architecture well covered, but a critical benchmark GAP confirmed Sept 12 2026**: no separate NAC license required (Cisco sells ISE + Duo/ZTNA as separate line items); native zero-trust integration; single policy engine across wired/wireless via Fabric Connect. Sources: "EN - Cisco Competitive Deck May 2026.pdf", "Competitive Overview - Cisco Competitive Positioning (12.18.24).pdf". **GAP: EN has no published RADIUS TPS / authentications-per-second benchmark anywhere in KB, while Cisco publishes exact per-appliance TPS tables (900-1300 TPS internal DB, 250-350 TPS AD/LDAP-backed, per SNS model) in its public "Performance and Scalability Guide for Cisco ISE."** Full apples-to-apples writeup (Cisco hardware specs, TPS tables, session-capacity tables, AD/LDAP degradation numbers, EN architectural differentiation, and the "why migrate without the numbers" argument) saved to `competitive_positioning/EN_NAC-CE_vs_Cisco_ISE_Benchmark_Analysis_20260912.md`. Neither vendor publishes a failover-time/RTO SLA — confirmed parity gap, not an EN-only weakness.
- **vs Aruba ClearPass — GAP, no dedicated battlecard found** in KB or SharePoint. Closest material: a third-party Gartner "Market Guide for Network Access Control" (June 2021) for independent landscape context, and the TTU transcript's partial Aruba-coexistence angle. A true head-to-head vs. ClearPass needs to come from Product Marketing/Competitive Intelligence directly if needed for a customer conversation.

## Highspot — accessed Sept 12 2026
OAuth completed; searched for ACE/NAC performance benchmarks, Cisco/Aruba battlecards. Reviewed "The Extreme Advantage" (Cisco competitive deck), ACE Management/Troubleshooting StudentGuides V26.1.0, Secure Network Fabric Sales Playbook EA v2. **Confirmed: no RADIUS TPS/ACE-throughput/failover-time benchmark and no dedicated Aruba ClearPass battlecard exist in Highspot either** — same gap as KB/SharePoint, now closed out as a genuine documentation gap (not a search-coverage issue). See `EN_NAC-CE_vs_Cisco_ISE_Benchmark_Analysis_20260912.md` Part 5.

## Bulk copy to local OneDrive folder (Sept 8 2026)
A background agent was tasked with copying the above documents from their source SharePoint sites into new subfolders (`demos_and_workshops/`, `competitive_positioning/`, `datasheets/`) under the existing `Extreme_Control_NAC` OneDrive folder, plus filling gaps in `deployment/` and `configuration/`. Results not yet folded in at time of writing this entry — check the live OneDrive folder directly, or ask this session for the copy-job status report.
