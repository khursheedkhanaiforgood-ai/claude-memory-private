---
name: project_l3vsn_grt_deepdive
description: L3 VSN (IPVPN) and GRT/IP Shortcuts deep-dive queued for next session, following Sep 5 2026 Fabric Connect teaching arc
type: project
originSessionId: 7403a6e9-ad49-464b-9285-c295b912c840
---
Next Fabric Connect session should go deeper on two topics only briefly covered Sep 5 2026:

1. **L3 VSN / IPVPN in detail** — how `router vrf` + `ipvpn` + `i-sid` + `ipvpn enable` actually redistributes routes between VRF instances on different BEBs; how it composes with L2 VSN (VRF's L2VSN VLANs each carry an IP gateway; the L3 VSN I-SID is the thing that ties those VRF instances together fabric-wide); inter-VSN routing mechanics.
2. **GRT / IP Shortcuts in detail** — `spbm 1 ip enable` (per-node, GRT-wide, not I-SID-consuming) vs. `redistribute direct` (only on the border-facing node that owns real external routes); IS-IS TLV 135/186 carrying GRT reachability; multicast (S,G) I-SID creation on BEBs for GRT multicast.

**Why:** Sep 5 session covered L2 VSN, 802.1aq/802.1ah frame anatomy, and BEB/BCB roles thoroughly, but user flagged L3 VSN and GRT as needing more depth before the live hardware lab (KhKLab-SW-01) proceeds further — directly relevant to the open I-SID 143154/144154 mismatch and border-node design questions already pending in `project_voss_fabric_migration.md`.

**How to apply:** When resuming Fabric Connect teaching, start here rather than re-deriving L2 VSN/802.1ah basics — those are done. Full frame/config walkthrough is in `EOD_HTML` repo (https://github.com/khursheedkhanaiforgood-ai/EOD_HTML) → `session_summary_20260905.html`.
