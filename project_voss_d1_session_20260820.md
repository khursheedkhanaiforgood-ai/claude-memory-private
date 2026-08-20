---
name: VOSS D1 Socratic Session — Aug 20 2026
description: Full D1 dialogue log for EOD HTML linking. Backbone fundamentals: MAC-in-MAC, IS-IS, BEB/BCB, B-FIB, I-SID boundary. Key breakthroughs and corrections captured.
type: project
originSessionId: aug-20-2026
---

## Session Context
Date: 2026-08-20
Topic: D1 — Backbone Fundamentals (Heart of VOSS)
Reference page: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/voss_migration_horizon.html#d1

---

## Key Concepts Mastered

### B-SA and B-DA (outer envelope)
- B-SA = ingress BEB's own B-MAC (derived from IS-IS system-id) — fixed, never changes in transit
- B-DA = egress BEB's B-MAC — looked up in B-FIB — set ONCE at ingress, immutable in transit
- C-MACs (client source/destination) sealed inside — invisible to core
- Breakthrough: "the outer loop never changes" → confirmed correct

### The Tunnel Analogy (user-generated)
"MAC-in-MAC is the highway with cars given a precise map of the exit. Exits are placed by SPB using Dijkstra. The exit is one-shot for the car."
- Validated. B-DA stamped at ingress, BCBs forward without rewrite, egress BEB strips envelope.
- Precision: it's L2 encapsulation not IP tunnel, but behavior is identical to a tunnel.

### The Two-Table Gap (key breakthrough)
User identified gap between IS-IS topology and service mapping.
- B-FIB = roads (how to reach a BEB) — built by IS-IS + Dijkstra
- I-SID Membership = services (which BEB hosts which I-SID) — built by RFC 6329 TLVs
- Service FIB (C-MAC table) = which specific BEB has this client — built by data-plane flood-and-learn
- "Know the roads, know the services" — user's synthesis

### Cross-I-SID Traffic
User asked: "what if AP_3000 on I-SID 100010 wants to reach laptop on I-SID 100020?"
Answer: I-SID is the L2 boundary. Fabric CANNOT cross I-SIDs at L2.
Must go L3 via Anycast Gateway SVI → inter-VLAN routing on BEB → ACL controls policy.

### Four FIBs
| FIB | Indexed by | Built by |
|---|---|---|
| B-FIB | B-MAC | IS-IS + Dijkstra |
| Service FIB (C-MAC table) | C-MAC per I-SID | Data-plane flood-and-learn |
| I-SID Membership | I-SID | RFC 6329 TLVs in IS-IS LSPs |
| IP FIB | IP prefix | IS-IS IP Shortcuts |

### Four Laws (final confirmed answers)
| Law | Standard | Role | Replaces |
|---|---|---|---|
| Foundation | IEEE 802.1aq | SPB — all-active paths, Dijkstra | STP |
| Brain | RFC 6329 | IS-IS extensions — B-FIB + I-SID table | OSPF + manual topology |
| Muscle | IEEE 802.1ah | MAC-in-MAC — immutable B-SA/B-DA | 802.1Q manual trunking |
| Handshake | IEEE 802.1Qcj | Fabric Attach — AP signals I-SID via LLDP | Manual port VLAN config |

### BEB vs BCB
- BEB = edge switch with UNI (client) + NNI (fabric) ports — does encap/decap
- BCB = core-only switch — no client ports, only forwards on B-DA, never sees C-MACs
- Single 5320 home lab = BEB only (no transit needed, no other fabric switches)
- Two 5320 lab = both are BEBs (each has client ports + NNI) — no BCB between them

---

## Corrections Made During Session
1. C-MAC ≠ switch MAC. C-MAC = client device MAC. Switch identity = B-MAC.
2. Handshake ≠ SVI. Handshake = 802.1Qcj Fabric Attach. SVI is the L3 Anycast Gateway interface.
3. SPB = 802.1aq (not RFC 6329). RFC 6329 = IS-IS extensions (Brain). User swapped them initially.
4. BCB exists BETWEEN BEBs — single switch lab has no BCB role.
5. B-DA is NOT next-hop rewrite (like traditional Ethernet). Set once at ingress, immutable.

---

## D1 Status
- All Q1-Q3 complete
- Four Laws: memorized with corrections
- Ready for D2: I-SID Service Model + Anycast Gateway
