---
name: EXOS → VOSS Rosetta Stone — Four Foundational Principles
description: Mental model for translating EXOS (VLAN-centric) configs into VOSS Fabric Engine (service-centric). Source: NotebookLLM/Alex synthesis May 7 2026. Use this as the framing for every EXOS→VOSS migration.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Source
NotebookLLM/Alex synthesis dialogue, 2026-05-07, in preparation for SW2 (KhKLab-SW-01) deployment with Corp + Guest VLANs via IPE → RDC Raleigh.

## The Big Shift
EXOS is **VLAN-centric**: you manually define every VLAN, every port tag, every uplink trunk, and every routing path. VOSS is **service-centric**: you define a *service* (an I-SID) and the Fabric figures out the rest.

The four principles are the mental hooks for every translation step.

## Principle 1 — The Brain (control plane)
| EXOS | VOSS |
|------|------|
| **STP / OSPF** — manually define every path; explicitly block redundant links to prevent loops. | **IS-IS** — silent control plane that builds a loop-free map of every possible path automatically. |

**Translation cue:** When EXOS config has explicit STP roles, OSPF areas, manual route maps — VOSS replaces ALL of that with one `router isis enable` block.

## Principle 2 — The Muscle (data plane)
| EXOS | VOSS |
|------|------|
| **802.1Q Trunks** — manually tag every VLAN on every uplink port. Add a VLAN to a trunk = config change on every switch. | **SPB (802.1aq)** — traffic is encapsulated in MAC-in-MAC ("suitcases") and moves across the fabric core automatically. |

**Translation cue:** EXOS uses VLAN tags AS the transport. VOSS uses SPB as transport, and VLANs ride INSIDE SPB. You don't manually tag uplinks — the I-SID handles that.

## Principle 3 — The Service (identity)
| EXOS | VOSS |
|------|------|
| **VLAN ID** — locally significant per switch; IDs must be manually aligned across all switches in the L2 domain. | **I-SID** — 24-bit global service ID that "stretches" services across the entire Fabric automatically. |

**Translation cue:** EXOS VLAN 70 on SW1 ≠ VOSS VLAN 70 on SW2 unless you manually align. **VOSS I-SID 100070 on SW1 == I-SID 100070 on SW2** — automatically. The I-SID is the service; the VLAN is just the local mapping.

## Principle 4 — The Edge (port provisioning)
| EXOS | VOSS |
|------|------|
| **Static Config** — ports hard-coded as "access" or "trunk" with specific VLAN tags. Adding an AP = manual port config. | **Auto-Sense / FA (Fabric Attach)** — ports detect devices (e.g., AP2) and dynamically provision the required services. |

**Translation cue:** EXOS port-3 on SW1 has hard-coded `tagged 20,30 untagged 1` for AP1. VOSS port 1/3 on SW2 has `auto-sense enable` + `fa enable` and the AP itself signals which VLANs/I-SIDs to provision. **The AP becomes the source of truth.**

## How to use the cheatsheet
For each line in the EXOS source config, ask: "Which principle does this belong to?" Then write the VOSS equivalent at the SAME principle level.

| EXOS pattern | VOSS pattern (same principle) |
|---|---|
| `enable stpd s0` | `router isis enable` (P1) |
| `configure stpd s0 add vlan X` | (no equivalent — IS-IS auto-discovers) |
| `configure vlan X tag Y` | `vlan create X type port-mstprstp 0` + `vlan i-sid X <isid>` (P3) |
| `enable ospf` | `router isis` + `ip-shortcut enable` (P1) |
| `configure vlan X add ports A,B tagged` | `auto-sense enable` + `fa enable` on ports A,B (P4) |
| `enable dhcp ports` | `ip dhcp-server subnet ... enable` (P3 + service) |

## Cross-references
- Memory: `reference_voss_cli.md` — confirmed VOSS CLI syntax (DHCP, ISIS, VLAN, FA)
- Memory: `reference_exos_cli_syntax.md` — EXOS syntax baseline
- Memory: `project_5320_new_arch_voss_ipe.md` — SW2 deployment context (Corp_New VLAN 70/I-SID 100070, Guest_New VLAN 80/I-SID 100080)