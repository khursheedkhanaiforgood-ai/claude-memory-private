---
name: FDB is authoritative VLAN truth — not ARP
description: show fdb ports X is the correct tool to verify which VLAN a client is on. ARP is historical (20-min TTL). When FDB and ARP disagree, always trust FDB.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
Use `show fdb ports <port>` to verify which VLAN a wireless client is on after a PPSK or policy change. Never rely on ARP alone.

**Why:** ARP entries persist for up to 20 minutes after a client moves to a different VLAN. The FDB (Forwarding Database) reflects the current live switching state. When a client's PPSK changes their VLAN assignment, the ARP table still shows the old VLAN entry. FDB shows where frames are actually forwarding RIGHT NOW.

**The verification sequence (always in this order):**
1. `show fdb ports 3` — which VLAN is this client's MAC currently in? This is truth.
2. `show iparp vlan VLAN1` and `show iparp vlan VLAN10` — confirms the IP the DHCP server assigned.
3. If FDB and ARP disagree → wait for ARP to age out (up to 20 min) or flush with `clear iparp`.

**Port-based vs VLAN-based FDB query:**
- `show fdb ports 3` — shows all MACs learned on port 3, grouped by VLAN. Best for AP ports: one command shows everything.
- `show fdb vlan VLAN10` — shows all MACs in that VLAN across all ports. Best when you already know the VLAN.

**Dual-MAC forensic observation (expected, not a bug):**
When testing both PPSK passphrases on the same device (e.g., Staff then Students on the same MacBook), you will see the same MAC address in TWO ARP tables simultaneously — one in `show iparp vlan Default` (old Staff entry, aging out) and one in `show iparp vlan VLAN10` (current Students entry). FDB will show the MAC only under VLAN10 (the current live state). This is expected behaviour from ARP TTL, not a misconfiguration.

**How to apply:** Any time a PPSK policy is deployed and VLAN steering needs verifying, go FDB first, ARP second. If a client reports unexpected VLAN assignment, FDB is the ground truth to reference before making any changes.

**Verified May 15 2026 — KB_School_AP_VLAN10_VLAN1:**
- `show fdb ports 3` showed MacBook MAC under VLAN10 after Students passphrase
- `show iparp vlan VLAN10` confirmed IP 10.10.0.11 (from SW1 DHCP)
- Stale ARP entry `192.168.0.10` in Default VLAN (from prior Staff test) present simultaneously — aged out within 20 min
