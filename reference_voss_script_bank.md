---
name: VOSS/FabricEngine CLI Script Bank
description: Verified VOSS FabricEngine CLI blocks organized by D1-D9 Socratic topics. Build as each session completes.
type: reference
originSessionId: 3db2d693-1e37-4eb5-8f10-fc421a5db74d
---
# VOSS FabricEngine CLI Script Bank
> Built session-by-session through D1–D9 Socratic revamp (Aug 2026)
> All commands: FabricEngine 9.3.x unless noted

---

## D1 — Backbone Fundamentals (MAC-in-MAC, IS-IS, BEB/BCB)

```bash
# View system B-MAC (the switch's backbone identity — its "zip code")
show isis system-id

# View IS-IS LSDB — the fabric's master directory
show isis lsdb

# View B-FIB — Dijkstra-derived forwarding table for B-MACs
show isis routes

# Show all IS-IS neighbors (adjacent BEBs/BCBs)
show isis neighbors

# Show SPB topology detail
show isis spb-topo

# Verify MAC-in-MAC encapsulation on a port
show fabric ports
```

---

## D2 — I-SID Service Model + Anycast Gateway

```bash
# Step 1: Create VLAN (local construct)
vlan create 50 name Corp type port-mstprstp 0

# Step 2: Bind VLAN to I-SID (fabric-wide service membership — TLV 185)
# Convention: I-SID = 100000 + VLAN ID (not enforced by protocol)
vlan i-sid 50 100050

# Step 3: Configure Anycast Gateway IP on VLAN SVI
# SAME IP must be configured on every BEB participating in this I-SID
interface vlan 50
  ip address 10.1.50.1/24
  ip forwarding

# Step 4: Enable IS-IS IP Shortcuts (carries L3 reachability across fabric)
router isis
  ip-source-address 10.1.50.1
  isis ip-shortcuts

# Verify I-SID membership advertised in LSDB
show isis i-sid

# Verify VLAN-to-I-SID binding
show vlan i-sid

# Verify IP interface on VLAN
show ip interface vlan 50

# Anycast MAC is system-derived from I-SID — view it with:
show vlan 50
# MAC shown = derived from I-SID 100050, identical on all BEBs with same I-SID
```

---

## D3 — Port Translation (coming next session)

---

## D4 — IP Shortcuts (coming)

---

## D5 — Failover / BFD (coming)

---

## D6 — XIQ Monitoring (coming)

---

## D7 — Alerts (coming)

---

## D8 — Alex's Milestone (coming)

---

## Key Reminders
- VLAN name = local only; I-SID number = fabric identity (must match across all BEBs)
- Anycast IP = same IP + same I-SID-derived MAC on ALL BEBs → no trombone on roam
- IS-IS IP Shortcuts required for L3 routing across the SPB fabric
- BFD + IS-IS = sub-second failover (~300ms); IS-IS alone = ~9s default
