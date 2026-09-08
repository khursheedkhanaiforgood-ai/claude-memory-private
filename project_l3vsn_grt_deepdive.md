---
name: project_l3vsn_grt_deepdive
description: L3 VSN (IPVPN) and GRT/IP Shortcuts deep-dive queued for next session, following Sep 5 2026 Fabric Connect teaching arc
type: project
originSessionId: 7403a6e9-ad49-464b-9285-c295b912c840
---
Next Fabric Connect session should go deeper on two topics only briefly covered Sep 5 2026:

1. **L3 VSN / IPVPN in detail — NOW SUBSTANTIALLY COVERED (Sep 7 2026), see new section below.** How `router vrf` + `ipvpn` + `i-sid` + `ipvpn enable` actually redistributes routes between VRF instances on different BEBs; how it composes with L2 VSN (VRF's L2VSN VLANs each carry an IP gateway; the L3 VSN I-SID is the thing that ties those VRF instances together fabric-wide); inter-VSN routing mechanics.
2. **GRT / IP Shortcuts in detail — GRT MECHANICS NOW COVERED (Sep 6 2026), see below.** IS-IS TLV mechanics, multicast (S,G) I-SID creation on BEBs for GRT multicast, and how GRT interacts with L3 VSN specifically, are still open.

**Why:** Sep 5 session covered L2 VSN, 802.1aq/802.1ah frame anatomy, and BEB/BCB roles thoroughly, but user flagged L3 VSN and GRT as needing more depth before the live hardware lab (KhKLab-SW-01) proceeds further — directly relevant to the open I-SID 143154/144154 mismatch and border-node design questions already pending in `project_voss_fabric_migration.md`.

**How to apply:** Both L3 VSN/IPVPN and GRT are now substantially covered (see sections below). Remaining open items are narrow: IS-IS TLV mechanics, multicast (S,G) I-SID creation for GRT multicast, and GRT/L3-VSN interaction specifics. Full frame/config walkthrough for L2 VSN/802.1ah is in `EOD_HTML` repo (https://github.com/khursheedkhanaiforgood-ai/EOD_HTML) → `session_summary_20260905.html`.

---

## L3 VSN / IPVPN — covered Sep 7 2026 (via separate transcript review)

Source: `Fabric-Connect-Lab-Full-Transcript_Sep 7 2026.pdf` (pages 30-41), a parallel Claude session the user ran outside this one, fully reviewed and folded in here.

- **VRF 0 = GlobalRouter = GRT** — built-in, never manually created. Reframes the whole topic from "GRT vs VRF" to "GRT vs L3 VSN": a VRF bound to an I-SID (`router vrf <name>` → `ipvpn` → `i-sid <number>` → `ipvpn enable`) *is* an L3 VSN. There's no separate "L3 VSN feature" — it's just what a non-zero VRF becomes once you give it an `ipvpn` I-SID.
- **Forwarding-plane distinction is one field**: GRT traffic uses the reserved system I-SID 0 implicitly (same mechanism as the GRT/IP-shortcuts material above); an L3 VSN's traffic carries the specific configured `i-sid` value instead. Everything else (B-MAC encapsulation, IS-IS-flooded reachability) is identical machinery.
- **Local `vrfid` values don't need to match across switches** — only the `i-sid` inside the `ipvpn` block does. The vrfid is a local index; the I-SID is the fabric-wide handle that ties VRF instances on different BEBs together.
- **Composition with L2 VSN**: a VRF's member VLANs (each an L2VSN with its own I-SID) each carry an IP gateway; the L3 VSN I-SID is the separate, additional binding that stitches those per-VLAN gateways into one fabric-wide routed domain. Isolation between I-SIDs is free/default; a VRF is only needed once a VLAN needs an IP gateway that must route somewhere.
- **Route-leak mechanism**: `router isis accept i-sid <X>` + `redistribute direct` — used to selectively leak routes between VRFs/GRT rather than full merge.
- **Scaling**: this is the same "16M I-SIDs vs. 4094 VLANs" property already documented in `reference_fabric_microsegmentation.md` — L3 VSN is what makes per-tenant/per-customer routed instances scale the same way L2 VSN already does.

**Open verification items**: none of this has been typed live on KhKLab-SW-01/SW2 yet — it's teaching-material-only, sourced from a parallel transcript, not a hands-on lab confirmation. Treat the mechanism descriptions as solid (consistent with the already-KB-verified GRT material) but verify actual `router vrf`/`ipvpn`/`i-sid` CLI syntax against `reference_voss_cli.md` or KB before typing it live.

---

## GRT / IP Shortcuts — self-study material reviewed Sep 6 2026

User independently produced two research documents outside this session (both reviewed and folded in here):

1. **`GRTs_Google Sep 5 2026.docx`** — a self-directed Google AI Mode Q&A transcript. Covers GRT vs. L2 I-SID as the two lanes a BEB can run simultaneously; a 3-tier BCB/BEB_Border/BEB_Edge config blueprint (hospital VLAN10 Corporate/GRT + VLAN20 Guest/I-SID example, extended with VLAN30 APs/GRT + VLAN40 WirelessUsers/I-SID); the "only the edge switch needs touching" property; and — most relevant to the live lab — a pivot into modeling the user's actual BEB_MyLab/SD-WAN/Fabric-Extend/Auto-NNI setup, including AP-DHCP troubleshooting (GRT lane needs `ip dhcp-relay`+`fwd-path` on the switch; L2 I-SID lane needs the relay configured on the **firewall**, not the switch) and a no-traffic-on-VLAN154 diagnostic sequence: `show isis spbm i-sid discover` → `show isis name-table` → `show isis spbm ip-unicast-fib`.
2. **`GRTs_Fabric_GoodDoc_Claude.pdf`** — a follow-up Claude session that **KB-verified** the Google doc's material against actual Fabric Engine 9.3 docs (User Guide, Troubleshooting Presentation, Best Practice deck, Command Reference) via Workato KB Search. This is the higher-trust source — treat it as superseding the Google doc wherever they conflict.

### Core mechanism (KB-confirmed)
Every switch (BCB and BEB) runs plain IS-IS/SPB, floods LSPs, and computes one shared B-MAC-only shortest-path tree — BCBs stop there, zero IP/VLAN/I-SID awareness. A BEB with `spbm 1 ip enable` + loopback + `ip-source-address` + `redistribute direct`(+`enable`+`metric`) + `isis apply redistribute direct` injects extra TLVs into those *same* LSPs, advertising owned subnets. Every other GRT-enabled BEB builds a second table (IP-unicast FIB: subnet → owning BEB) from this — same flooding mechanism, no separate protocol ("IP shortcuts" = piggyback routing on the existing LSDB). Unknown/external destinations resolve via BEB_Border's `redistribute static` for `0.0.0.0/0`, flooded the same way (ordinary longest-prefix-match over IS-IS).

**Correction to item 2's original framing above:** `redistribute direct` is NOT border-only — every GRT-enabled edge BEB needs it (so its local subnets become reachable to other BEBs). Only `redistribute static` (for the default route) is border-only.

### Two worked traffic-flow exercises
- **BEB_X → YouTube**: FIB miss → falls to BEB_Border's default route → packet encapsulated with outer B-DA = BEB_Border's B-MAC and the **reserved system GRT I-SID** (not a manually configured one) → BCBs forward blind on B-MAC → BEB_Border strips header, routes out its static default to firewall.
- **BEB_X → BEB_Y**: exact subnet match (BEB_Y's own redistributed route) → same encapsulation, direct edge-to-edge, never touches BEB_Border/firewall — this is the actual "shortcut."

### Three-tier config blocks (KB-cited, corrects the Google doc's syntax gaps)
- **BCB**: `vlan create 4051/4052 type spbm-vlan` (Google doc omitted this step) → `router isis` block (`spbm 1`, `spbm 1 b-vid 4051,4052 primary 4051`, `system-id`, `spbm 1 nick-name`, `manual-area`) → `router isis enable` → NNI ports either explicit (`isis` / `isis spbm 1` / `isis spbm 1 l1-metric` / `isis enable`) or `auto-nni` (+`isis l1-metric` only — no explicit `isis spbm 1` needed, corrects Google doc). Zero L3, zero I-SID, ever.
- **BEB**: everything above, plus `interface loopback 1` with `/32` IP, `router isis` → `ip-source-address`, `spbm 1 ip enable`, `redistribute direct`/`redistribute direct enable`/`redistribute direct metric 1`, then top-level `isis apply redistribute direct`; plus a local customer VLAN whose subnet is what gets redistributed.
- **BEB_Border**: a BEB, plus firewall-transit VLAN/IP, `ip route 0.0.0.0 0.0.0.0 <fw-ip> weight 1`, `router isis` → `redistribute static`/`redistribute static enable`, top-level `isis apply redistribute static`.

### Open verification items (flagged by the KB-verified doc itself, not yet confirmed on-switch)
- The `redistribute static`/`isis apply redistribute static` pairing for BEB_Border's default route is confirmed as *generically valid* syntax (KB confirms `static` as a valid redistribution source, confirms the identical `direct` pattern verbatim) but **not** verified as a verbatim border/firewall example — applied by analogy. Verify with `show ip isis redistribute` and `show ip route` before trusting it live.
- **Static-route syntax conflict with our own switch-confirmed reference**: `reference_voss_cli.md` (verified live on the actual 5320, Apr 30 2026) shows a **two-step, CIDR-format** static route: `ip route 0.0.0.0/0 <gw> weight 1` then a separate `ip route 0.0.0.0/0 <gw> enable`. The PDF's BEB_Border block instead uses a **single-line, dotted-mask** form with no separate enable step. Since the PDF itself flags this block as unverified, trust `reference_voss_cli.md`'s syntax if actually typing this on KhKLab-SW-01.

**How to apply:** GRT mechanics + config blocks above are now solid enough to teach from directly. Next live session should (a) start L3 VSN/IPVPN from scratch, and (b) if/when applying GRT config on KhKLab-SW-01, use `reference_voss_cli.md`'s confirmed static-route syntax rather than the PDF's unverified form.
