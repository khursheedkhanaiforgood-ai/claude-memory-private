---
name: project_nac_ce_vs_cisco_client_prep
description: Client meeting Tuesday needs empirical EN_NAC-CE vs Cisco ISE benchmark data (QoE/scalability/security) — TPS/failover benchmark gap CONFIRMED across KB+SharePoint+Highspot, document now fully finalized (Sept 13, incl. KPI benchmarking + live sources + dialogue log), escalation to PM/GTAC is the only remaining path
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

**Sept 13 continuation — document now fully closed out, all three formats (.md/.html/.docx) in sync:**
6. Corrected an inaccurate claim (Section 3a) that Cisco does "continuous per-packet inline enforcement" — both vendors actually use RADIUS CoA post-auth (Cisco via pxGrid/ANC/SGT, EN via Quarantine-policy/NGFW-SIEM); new Section 3f documents the real threat-response comparison.
7. Added "Part 6 — Industry-Standard KPI Benchmarking": named frameworks (ISO/IEC 25010, ITIL, NIST 800-207, Gartner 5-yr TCO, Forrester TEI, Gartner NAC Market Guide) applied to a 10-row KPI table. **Key caveat for Tuesday: Cisco's 191% ROI (Forrester TEI, Cisco-paid) and Extreme's 32% TCO reduction (ACG Research, Extreme-commissioned) are NOT apples-to-apples — different products/scope/baseline — do not put them on the same slide without that caveat.**
8. Added a "Master Sources List" — every source in the whole document (not just Part 6) now has a live public URL where one exists (Cisco Perf & Scalability Guide, Extreme vs Cisco Catalyst, ExtremeControl/Network Security/Fabric Connect/Site Engine product pages); internal-only KB/Highspot items explicitly flagged as having no public URL rather than fabricated.
9. Added "Appendix A — Session Dialogue Log" — full Q&A trace (verbatim questions + summarized answers) so every conclusion in the document is traceable to the question that produced it, per explicit user request.
10. EOD summary for this closeout: `~/lab-journal/session_summary_20260913.html` (pushed to `EOD_HTML` repo, commit `575d70d`).
