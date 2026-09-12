---
name: project_nac_ce_vs_cisco_client_prep
description: Client meeting Tuesday needs empirical EN_NAC-CE vs Cisco ISE benchmark data (QoE/scalability/security) — TPS/failover benchmark gap CONFIRMED across KB+SharePoint+Highspot, document finalized, escalation to PM/GTAC is the only remaining path
type: project
originSessionId: 7403a6e9-ad49-464b-9285-c295b912c840
---
User has a client meeting Tuesday (week of Sept 12 2026) where the client is expected to press on empirical performance/scalability/security metrics for EN_NAC-CE (ExtremeControl/Access Control Engine), because a competitor (implied Cisco) advertises QoE-mapped metrics directly: concurrent authentications/sec, re-authentications/sec, CE/NAC failover time, and how the LDAP/AD path constrains a "100K end-devices, 200 IOPS" style sizing claim.

**Why:** User explicitly does not want to hand-wave in front of the client — wants empirical/R&D-grade evidence or an honest, defensible architectural answer instead of invented numbers.

**Key finding (Sept 12, confirmed via Workato KB Search + live fetch of Cisco's public Performance and Scalability Guide):**
- Cisco publishes hard RADIUS TPS tables and session-capacity tables per SNS appliance model. EN has no equivalent published rate/throughput benchmark anywhere in the internal KB — only capacity (end-systems trackable) numbers, e.g. ACE tiers Small 3K/Medium 6K/Enterprise 9-12K/Large Enterprise 12-24K end-systems, and UCP 2130C 200K ExtremeControl end-systems.
- Neither vendor publishes a failover-time/RTO SLA — genuine parity point, not an EN-only gap.
- Cisco's own numbers show AD/LDAP-backed auth cuts TPS by ~72% vs internal DB on the same hardware — useful, honest talking point since it proves the AD/LDAP constraint is universal, not EN-specific.
- Full analysis with tables, architectural differentiation (why EN's Fabric-distributed-enforcement model matters more than raw TPS), and the "why migrate without the numbers" argument saved to: `~/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Documents/Extreme_Product_Reference/Extreme_Control_NAC/competitive_positioning/EN_NAC-CE_vs_Cisco_ISE_Benchmark_Analysis_20260912.md`.

**How to apply:** Before Tuesday:
1. **[DONE, Sept 12]** User completed Highspot OAuth; searched "The Extreme Advantage" Cisco competitive deck, ACE Management/Troubleshooting StudentGuides V26.1.0, Secure Network Fabric Sales Playbook EA v2, and a direct "ACE sizing/RADIUS TPS" query — **no internal RADIUS TPS, ACE throughput, or ACE-engine failover-time benchmark exists anywhere in Highspot either.** The gap is now confirmed across all three internal channels (KB, SharePoint, Highspot), not a search-coverage artifact.
2. **No internal TPS/failover-SLA data exists** — escalate to Product Management/GTAC directly if the client demands a hard number; do not fabricate one.
3. Recommended client-facing framing: pivot from "raw TPS" to "where enforcement work happens" (EN pushes enforcement to Fabric edge silicon post-auth; Cisco's ISE/PSN stays more continuously in the data path, hence why their TPS ceiling matters more to them) plus TCO/operational-complexity argument (Cisco's N+1 PSN + dual load balancer + DNA Center sync requirement vs EN's single-policy-engine/native-Fabric-integration model).
4. **Aruba ClearPass battlecard gap also confirmed closed (absence verified) in Highspot** — only tangential HPE-Wired and Secure Network Fabric hits surfaced, no dedicated ClearPass battlecard. Same "no benchmark, no battlecard" pattern as Cisco — worth raising as one bundled ask to Product Marketing/Competitive Intelligence rather than one-off asks across deals.
5. Final document, fully updated with Highspot findings: `EN_NAC-CE_vs_Cisco_ISE_Benchmark_Analysis_20260912.md` (Part 5 rewritten Sept 12 to reflect confirmed-absent status, not pending search).
