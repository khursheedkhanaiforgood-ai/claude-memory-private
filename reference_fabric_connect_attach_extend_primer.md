---
name: reference_fabric_connect_attach_extend_primer
description: Detailed architecture primer on Fabric Attach, Fabric Connect, and Fabric Extend — mechanisms, standards, control/data plane split, B-MAC/System-ID identity, and how they relate to each other
type: reference
originSessionId: b60a6aab-a768-4fff-b217-9bd45e71c887
---
KB-verified deep dive (Sept 2 2026 session), building on the VOSS/Fabric migration project thread (`project_voss_fabric_migration.md`).

## Fabric Attach (FA)

Two-phase mechanism, both riding on LLDP:

**Phase 1 — Discovery.** FA Client (or FA Proxy, for non-SPB devices) and FA Server exchange LLDP PDUs carrying FA-specific organizational TLVs. Finds/identifies both sides over the local link.

**Phase 2 — Signaling (the actual auto-provisioning).** Client/proxy requests a VLAN(C-VID)-to-I-SID mapping over that same LLDP channel. KB: *"the FA Server accepts requests... to add the C-VID and I-SID elements in the SPB network, and also automatically configures the necessary C-VID and I-SID"* — server acks success/fail.

**Three roles:**
- **FA Server** — the fabric switch (a BEB). Does the actual provisioning.
- **FA Client** — a device that speaks FA directly (some APs).
- **FA Proxy** — sits in front of non-FA devices, does the handshake on their behalf, aggregates multiple downstream clients via a Switched-UNI (S-UNI) over a plain 802.1Q-tagged uplink.

**Scope limit:** FA only auto-provisions **L2VSNs**. KB: *"L3VSNs and Multicast Virtualization for L2/L3 must be added and pre-configured at the FA Server"* — routed/multicast services still need manual config.

**Confirmed elsewhere this session:** FA is edge-device-to-BEB only — never BEB-to-BEB. LLDP is local-link only — does not traverse a Fabric Extend WAN tunnel (see FE section, ZTF note).

## Fabric Connect

Umbrella architecture. Standards-based:
- **IEEE 802.1aq** (Shortest Path Bridging) / **RFC 6329** (IS-IS extensions for SPB) — **control plane**.
- **IEEE 802.1ah** (Provider Backbone Bridging, MAC-in-MAC) — **data plane / envelope format only**.

**Common misconception, corrected this session:** 802.1ah does NOT compute anything — it's just the outer header format. **802.1aq/IS-IS is what "figures out the shortest path,"** and it does so continuously/ahead of time (standing state), not per-packet. BCBs forward by simple table lookup on a table IS-IS already built and keeps current.

**B-MAC = node identity, derived from System ID.** KB: *"The System ID becomes the BMAC for the Node"* and *"the Backbone MAC is used as both source and destination MAC address in the SPBM network."* Each switch has exactly one B-MAC. The destination B-MAC in any frame identifies the egress node — i.e., **the terminating B-MAC IS the BEB where the VSN/service actually lives.** Nicknames are conventionally derived from System ID by schema (e.g. System ID `020s.ssnn.0000` → nickname `s.ss.nn`).

**Corrected end-to-end sequence:**
1. IS-IS (802.1aq) has already computed shortest-path trees to every node's B-MAC — standing state, not triggered per-frame.
2. Traffic enters at ingress BEB1. BEB1 wraps it in an 802.1ah header: src B-MAC = BEB1, dst B-MAC = BEB2 (egress node), I-SID = the service.
3. Each BCB in the path does a table lookup on that destination B-MAC (using IS-IS's precomputed table) and forwards — no path computation at forward-time.
4. Egress BEB2 recognizes the destination B-MAC as its own, strips the 802.1ah header, reads the I-SID, hands the inner frame to the right local VSN/C-VLAN/UNI port.

**Who-knows-what split (why the core needs zero per-service config):** KB: *"Only the BEB has knowledge of the L3 VSN and corresponding IP/ARP/MAC addresses. The BCB only has knowledge of each Backbone MAC address (B-MAC) used to send traffic across an SPB network."* This is a structural/visibility fact, not a policy choice — the BCB literally cannot see services, only B-MAC reachability. Ties directly to the earlier-established "BCB vs BEB is a config role, not hardware" finding: role = whether any I-SID is bound to that node, not the node's capability.

## Fabric Extend (FE)

Stretches the Fabric Connect model above across an IP WAN instead of a physical NNI.

**Use cases (KB):** remote/branch connectivity, campus-to-DC, DC interconnect, connecting separate "Fabric islands" — while keeping **one operational model** across campus/DC/branch.

**Business case, direct quote:** *"reduce operating costs by managing branch, campus, and data center as a single entity... instant onboarding and dynamic auto-attach... automates service provisioning so services are provisioned only at source and destination rather than hop-by-hop."*

**Transport:** VXLAN over private/public IP WAN; optional IPsec wrap when crossing open internet (requires Fabric IPsec Gateway VM on supported platforms — 5320 explicitly excluded from FE-over-IPsec, confirmed via Fabric Engine 9.3 User Guide Table 120, logged in `project_voss_fabric_migration.md`).

**One tunnel, many services:** *"multiple logical segments inside a single VXLAN tunnel"* — one FE tunnel carries many I-SIDs simultaneously, same isolation as physical NNI.

**WAN-agnostic:** swapping ISPs / WAN providers doesn't touch fabric config, only the IP transport underneath.

**Both-ends-required, confirmed exact CLI (this session):**
```
ip-tunnel-source-address <local-IP> vrf fe
logical-intf isis <1-255> dest-ip <peer-IP> mtu <value> [name WORD]
isis
isis spbm <1-100>
isis enable
```
Mirrored on the peer switch with source/dest swapped. KB: *"logical ISIS interface adjacency requires configuration on both sides."*

**ZTF/Auto-sense does NOT extend across the FE tunnel** — LLDP (which drives ZTF) is local-link only. ZTF/Auto-sense DOES apply on a genuine local physical NNI (e.g., two BEBs directly cabled together) — confirmed via KB: *"if you start two nodes in a network without an existing configuration file, then Zero Touch Fabric Configuration dynamically establishes an IS-IS adjacency between them"* (B-VLAN dynamically learned from neighbor over that local link).

**Auto-Sense/SD-WAN-IPE variant:** on the port facing an IPE, `auto-sense enable` lets LLDP-based automation build the FE tunnel/logical-IS-IS-interface automatically (`show interfaces gigabitethernet auto-sense` reports state `SD-WAN` once detected) — a separate, IPE-specific automation path, not standard Fabric Attach.

**Open/unconfirmed items (flagged, not resolved):**
- MTU: earlier claim of ≥1594-byte floor vs. Extreme's own reference config using `mtu 1500` on both sides — never reconciled.
- Whether FE-over-SD-WAN's IPE-encrypted underlay lets the 5320 sidestep its native FE-over-IPsec restriction.
- Whether the XA1400 appliance is required or optional for a pure dedicated-circuit VXLAN FE design (no SD-WAN overlay).
- Whether an FE tunnel can terminate directly on a bare BCB (no I-SID bound) — KB shows the only confirmed I-SID/service-gateway role is BEB, and Extreme's own SD-WAN reference topology labels the hub FE endpoint "BEB/BCB" (combined), suggesting in practice it's paired with BEB function, not proven either way as a hard requirement.

Full narrative/decision log for the live POC (Karl's Core_BCB, HQ_BEB, Remote_BEB, IPEs, internet egress mechanics) lives in `project_voss_fabric_migration.md`.
