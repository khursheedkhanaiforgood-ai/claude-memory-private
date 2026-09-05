---
name: reference_fabric_microsegmentation
description: Extreme Fabric Connect "hyper-segmentation"/microsegmentation + "Native Stealth" architecture — VSN/I-SID mechanism, IP-invisible topology, how it differs from VLAN+ACL/firewall segmentation, Fabric Attach role, Dynamic Workgroups/Identity Engines/Senetas extensions
type: reference
originSessionId: b60a6aab-a768-4fff-b217-9bd45e71c887
---
Sources: `/Users/khukhan/Downloads/11909-Smart-Networking-Hyper-Segmentation-SB_v6.pdf` + `/Users/khukhan/Downloads/11908-Smart-Networking-Native-Stealth-SB_v5.pdf` (matched-pair Extreme solution briefs, 2019) + Workato KB cross-check (2026-09-01).

**Core mechanism:** Fabric Connect segments traffic into Virtual Service Networks (VSNs), each identified by a Service ID (I-SID) in the IEEE 802.1ah MAC-in-MAC header. 24-bit ID space = 16M+ possible segments vs. ~4,094 for VLANs. This is the exact same I-SID mechanism already used in the VOSS/Fabric Connect home lab (`project_voss_fabric_migration.md` VLAN→I-SID map) — the lab's VLAN 10/20/30/50/60 → I-SID 100010/100020/etc. mapping IS Extreme's microsegmentation architecture in practice, just not previously labeled as such.

**Why it differs from VLAN+ACL/firewall segmentation:**
- Conventional networks are inherently any-to-any/routed by default; ACLs and firewalls are bolted on afterward to restrict paths — expensive, complex to plan/maintain.
- Fabric Connect is isolated-by-design: VSNs are "oblivious to each other" ("ships-in-the-night") — no default reachability to restrict. Traffic is encapsulated at the Edge only and opaque to intermediate/core nodes.
- Edge-only provisioning: config for a service only exists on the nodes where that service is actually present — core/intermediate nodes need zero per-service config, unlike hop-by-hop VLAN tagging.
- Hyper-segmentation is complementary to (not a replacement for) firewalls/IDS/defense-in-depth — narrows the traffic baseline so anomaly detection is more precise.

**Fabric Attach's role:** auto-attaches authenticated endpoints directly into their correct VSN at the edge (Wiring Closet or DC) — this is the same FA mechanism already used for AP1/AP3000 onboarding in the home lab.

**Capabilities beyond current lab scope (flagged, not yet built):**
- **Dynamic Workgroups** — self-provisioned, on-demand private network segments for teams/projects (e.g., M&A "Chinese Wall" use case), triggered via workflow (Extreme Breeze), with optional end-to-end encryption via a Senetas partnership.
- **Identity Engines integration** — fine-grained authN/authZ as the policy enforcement point before any VSN connectivity is granted (ties to the UZTNA/RADIUS Filter-Id work already done in the EXOS UZTNA thread).

**Note:** source PDF is from 2019 — Extreme's newer material (per KB, "Why Microsegmentation Is Key to Enhancing Network Security" blog) uses "microsegmentation" directly rather than the "hyper-segmentation" branding, but describes the identical VSN/I-SID mechanism.

## Native Stealth (companion brief — same VSN/I-SID substrate, security-through-obscurity lens)

**IP-invisibility:** Fabric Connect is Ethernet-centric, not IP-centric. Forwarding rides the MAC-layer VSN-ID (I-SID, 802.1ah header) + destination Edge Bridge MAC — no hop-by-hop IP path exists for IP-based recon/mapping tools to trace. Conventional networks are inherently mappable via routing tables; Fabric Connect has no equivalent IP path to expose.

**Visibility capped at the VSN edge:** a host can see, at most, other hosts on its own VSN — nothing else. Even if ICMP is enabled, it would only reveal the VSN's own Edge nodes, never the inner fabric/core nodes. This is stronger than "segments can't reach each other" (the micro-seg framing) — it's "the fabric interior is structurally invisible even to permitted traffic."

**Fabric Attach lifecycle (security framing):** FA config isn't just auto-provisioned on connect — it's automatically torn down when the session ends. A stale VLAN/policy binding left behind after a device disconnects is itself an attack surface; ephemeral provisioning removes that surface as soon as the session ends, not just as a convenience/scale feature.

**Maps onto the home lab:** `show isis lsdb` (topology) and `show fa assignment` (ACTIVE/DYNAMIC) are exactly these mechanisms in practice — the LSDB is only ever visible switch-to-switch inside the fabric, never exposed via IP tools from a host.

## ZK Research validation (3rd source, independent analyst — Dec 2024)

Source: `/Users/khukhan/Downloads/Extreme Networks Extreme Fabric report - REVISED.pdf` — Zeus Kerravala/ZK Research white paper, "Network Fabrics: The Foundation for Scaling and Securing Today's Distributed Enterprises," 13 pp., commissioned but analyst-authored (independent third-party validation, not vendor marketing copy like the two 2019 solution briefs).

**The key new fact: this ties the two prior PDFs together as one named pillar.** Extreme's current (2024) architecture is framed as three principles — **Unify / Automate / Secure** — and "Secure" is defined as having exactly two components: **automated micro-segmentation** and **stealth topology**. In other words, 11909 (hyper-/micro-segmentation) and 11908 (native stealth) aren't two separate features — they are literally the two named halves of Extreme's "Secure" pillar, confirmed by an independent analyst reading the current architecture, not just two matched-vintage 2019 marketing docs.

**Corroboration, now from a different angle:**
- Micro-segmented services are explicitly described as being **"spun down"** when no longer needed — same lifecycle-teardown property as Fabric Attach's session-end cleanup in the stealth doc, but here attributed directly to micro-segmentation rather than just FA.
- **SPB convergence: ~200ms** (sub-second) — quantifies the "Automate" pillar's resiliency claim; relevant context for why the fabric can auto-heal without operator intervention.

**SPB vs. IP fabrics (EVPN/VXLAN) — new framing not present in the solution briefs:** ZK Research treats these as the two variants of "network fabric" in the industry today. SPB (802.1aq, IS-IS control plane) is positioned as better suited to enterprise campus — simpler, more flexible, zero-touch — while IP/EVPN-VXLAN fabrics are positioned as the carrier/L3-data-center-oriented approach. This is useful vocabulary for placing VOSS/Fabric Connect within the broader industry landscape (and relevant background for the paused Fabric Extend thread, since FE's VXLAN-tunnel option is literally borrowing the "IP fabric" transport to extend an SPB fabric).

**Adoption model — mirrors the user's own lab philosophy:** the paper stresses "adopt fabric at your own pace" — campus-core-only, or campus+data center, or just specific edge segments — no forklift/all-or-nothing requirement. This is the same incremental logic already in use for the paused FE thread's Test 1/2/3 staged lab-build plan.

**Concrete proof points (named customers):** SDCCD (San Diego Community College District), DWTC (Dubai World Trade Centre), ADRZ — cited as real-world deployments, not just capability claims.

**Net effect on this reference file:** with all three sources now cross-read, "Secure = micro-segmentation + stealth topology" is confirmed as Extreme's own current architectural taxonomy, independently validated, and both underlying mechanisms (VSN/I-SID scale + IP-invisible topology) map 1:1 onto commands already run in the home lab.
