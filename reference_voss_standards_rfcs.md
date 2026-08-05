---
name: VOSS Architecture — Standards & RFCs Reference
description: The 5 IEEE/IETF standards that define VOSS Fabric Engine (SPB, IS-IS extensions, PBB, FA, CFM). Maps each standard to the 4-principle Rosetta Stone. Source: NotebookLLM/Alex synthesis 2026-05-07.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Why this file
The 4-principle cheatsheet (`reference_exos_voss_4_principles.md`) explains *what* VOSS does differently. This file explains *why* — the underlying IEEE / IETF standards. When you're reading docs, debugging, or talking to vendors, these are the names everyone uses.

## The 5 Pillars

| Standard | Role | Definition |
|---|---|---|
| **IEEE 802.1aq** | The Foundation (P2: Muscle) | **Shortest Path Bridging (SPB)** — the primary standard enabling a loop-free, multi-path Ethernet mesh. Replaces traditional Spanning Tree (STP). |
| **RFC 6329** | The Brain (P1) | **IS-IS Extensions for SPB** — defines how the IS-IS link-state protocol builds network topology and distributes service info (I-SIDs) without needing an IP stack. The control plane that makes 802.1aq work. |
| **IEEE 802.1ah** | The Muscle / Data Plane (P2) | **Provider Backbone Bridging (PBB)** — MAC-in-MAC encapsulation. User traffic gets wrapped in a backbone header, hiding device MACs from the core for scalability. The "suitcase" model. |
| **IEEE 802.1Qcj** | The Edge (P4) | **Fabric Attach (FA)** — automates the connection between non-fabric devices (e.g., AP3000) and the Fabric. Edge devices dynamically request/join services via LLDP. |
| **IEEE 802.1ag** | Diagnostics / OAM | **Connectivity Fault Management (CFM)** — heartbeats, traceroute, fabric path optimization. The "show ping/traceroute" of SPB. |

## Mapping to the 4 Rosetta Stone Principles

| Principle | Standards involved | What "owns" the principle |
|---|---|---|
| 1. Brain (control plane) | **RFC 6329** + IEEE 802.1aq | IS-IS auto-builds the loop-free topology and distributes I-SIDs. |
| 2. Muscle (data plane) | **IEEE 802.1aq** + IEEE 802.1ah | SPB picks the path; PBB does the MAC-in-MAC encapsulation. |
| 3. Service (identity) | I-SID (24-bit, defined within 802.1ah) | Range 1 → 16.7M. Globally unique, recognized by every switch in the Fabric. |
| 4. Edge (port provisioning) | **IEEE 802.1Qcj** | FA: AP3000 uses LLDP to say "I'm an AP, I need I-SID 100070 for my SSID" → switch plumbs it on demand. |
| (Diagnostics, cross-cutting) | **IEEE 802.1ag** (CFM) | Used for `ping isis`, `traceroute isis`, fabric reachability checks. |

## Why each standard matters in practice

**IEEE 802.1aq — SPB**
Replaces STP entirely. STP picks ONE path and blocks redundant ones (active-passive). SPB picks the SHORTEST path per source-destination pair (active-active across all uplinks). No more wasted capacity.

**RFC 6329 — IS-IS for SPB**
This is what makes SPB *automatic*. Engineers don't draw the topology — IS-IS does. When a new switch joins the fabric, every other switch learns about it via LSP (Link State PDU) flooding within seconds. That's why VOSS configurations look so much shorter than legacy switch configs.

**IEEE 802.1ah — PBB (MAC-in-MAC)**
Customer MAC (`C-MAC`) is wrapped inside a Backbone MAC (`B-MAC`) header. Core switches only learn B-MACs (one per backbone edge), not customer MACs. This is what lets the fabric scale to 100,000s of clients without exploding the FDB.

**IEEE 802.1Qcj — Fabric Attach**
The "service-at-the-edge" magic. Without FA, an AP would need to be manually configured with VLAN tags matching every SSID's VLAN. With FA, the AP advertises *which I-SIDs it needs* via LLDP TLVs, and the switch's FA module dynamically provisions the port. That's how `auto-sense enable` + `fa enable` on a single port replaces ~10 lines of manual VLAN tagging.

**IEEE 802.1ag — CFM**
The often-forgotten diagnostics layer. Without CFM, you have no fabric-aware ping/traceroute — only IP-layer tools. CFM lets you verify SPB reachability between any two switches even if IP routing isn't working.

## Useful in conversations / docs
When reading Extreme docs, vendor papers, or talking to support:

- "SPB" → 802.1aq
- "IS-IS for SPB" → RFC 6329
- "MAC-in-MAC" / "PBB" / "B-MAC" / "C-MAC" → 802.1ah
- "FA" / "Fabric Attach" → 802.1Qcj
- "CFM" / "Y.1731" / "OAM" → 802.1ag

## Cross-references
- `reference_exos_voss_4_principles.md` — the mental model (Brain/Muscle/Service/Edge)
- `reference_voss_cli.md` — verified CLI syntax
- `reference_voss_command_themes.md` — 4-lens command index
- `project_5320_new_arch_voss_ipe.md` — SW2 deployment context