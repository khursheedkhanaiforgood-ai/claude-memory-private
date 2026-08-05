---
name: KB_School Sprint — May 14 2026
description: New clean-slate lab: SW1 factory reset, XIQ policies KB_School_SW1 + KB_School_AP1_VLAN10, SSID KB_School_Broadcast, PPSK students/staff on VLAN10. AP1 CAPWAP established May 14.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Goal
Build KB_School lab from scratch — two clean XIQ policies (SW1 + AP1), PPSK SSID with two user groups (students/staff), both landing on VLAN10. Practice for boss Karl's VIQ deployment.

## Why
Previous VLAN100 sprint had persistent XIQ Supplemental CLI failures. Fresh start with clean factory resets of both SW1 and AP1. Lesson learned: VLAN10 naming (no underscore) and port type architecture.

## Devices
- SW1: 5320-16P-2MXT-2X, serial FJ012544G-00233, IP 192.168.0.28
- AP1: AP3000, serial HA012519Y-10623, MAC 00:E6:0E:55:66:80

## Policy: KB_School_SW1
- Type: Switching/Routing
- VLAN: VLAN10 (tag 10)
- SVI: 10.10.0.1/24, IPv4 Forwarding ON
- Port 1: Trunk (Default VLAN, uplink to QF-Modem)
- Port 3: Default VLAN untagged (AP management) + VLAN10 tagged (client traffic)
- DHCP pool: 10.10.0.10 - 10.10.0.254, gateway 10.10.0.1, DNS 8.8.8.8/1.1.1.1, lease 86400
- Supplemental CLI name: uses `VLAN10` (verified correct name on switch)

## Policy: KB_School_AP1_VLAN10
- SSID: KB_School_Broadcast
- Auth: PPSK, WPA2, CCMP
- User Groups: Staff_PPSK, Students_PPSK (both SERVICE/cloud)
- Default User Profile: VLAN10 → all users land on VLAN10
- AP Template: AP_3000-default-template

## VLAN10 Supplemental CLI (CONFIRMED WORKING)
```
configure vlan VLAN10 dhcp-address-range 10.10.0.10 - 10.10.0.254
configure vlan VLAN10 dhcp-options default-gateway 10.10.0.1
configure vlan VLAN10 dhcp-options dns-server primary 8.8.8.8
configure vlan VLAN10 dhcp-options dns-server secondary 1.1.1.1
configure vlan VLAN10 dhcp-lease-timer 86400
enable dhcp ports 3 vlan VLAN10
```

## Key Lesson — AP Port Architecture (May 14)
**WRONG:** VLAN10 untagged on port 3 (AP management on 10.10.0.x — can't reach XIQ)
**CORRECT:** Default VLAN untagged on port 3 (AP management 192.168.0.x → QF-Modem → XIQ) + VLAN10 tagged on port 3 (client user traffic)

| VLAN | Port 3 Type | Purpose | DHCP from |
|------|-------------|---------|-----------|
| Default (1) | Untagged | AP management → XIQ via QF-Modem | QF-Modem (192.168.0.x) |
| VLAN10 | Tagged | Client user traffic (PPSK assigned) | SW1 (10.10.0.x) |

XIQ pushed VLAN10 as untagged — wrong. Fixed via CLI:
```
configure vlan VLAN10 delete ports 3
configure vlan Default add ports 3 untagged
configure vlan VLAN10 add ports 3 tagged
save config
```

## XIQ Supplemental CLI Lessons (from both VLAN100 + VLAN10 sprints)
1. `${vlan:NAME}` resolves to VLAN ID number → EXOS rejects. Always hardcode name.
2. Check exact VLAN name on switch with `show vlan` AFTER first deploy
3. XIQ VLAN naming: no underscores (VLAN10, not VLAN_10; Guest100, not Guest_100)
4. Deploy WITHOUT Supplemental CLI first → verify base config → add CLI back
5. 15% deploy hang = Supplemental CLI failing, not a network issue

## Factory Reset Lessons (May 14)
- EXOS factory reset: `unconfigure switch all` (NOT delete config.cfg)
- After factory reset, provisioning wizard runs — say NO to "disable unconfigured ports"
- Always run `enable ports all` + `save config` immediately after factory reset
- Default login after factory reset: admin / (blank password)

## SW1 Final Config Verification (show vlan)
```
Default: 192.168.0.28/24, ports 1-20 untagged (2 active: port1 + port3)
VLAN10:  10.10.0.1/24, ipforwarding ON, port 3 tagged (1 active)
```

## Status (May 14) — COMPLETE
- SW1 factory reset + policy deployed: ✅
- Supplemental CLI (DHCP): ✅ WORKING
- AP1 factory reset: ✅
- AP1 CAPWAP established (LED green): ✅
- AP1 XIQ updated + SSID broadcasting: ✅
- Client PPSK test (MyLab): ✅ COMPLETE — iPhone 10.10.0.10, MacBook 10.10.0.11, internet working
- Karl's VIQ deployment: ✅ COMPLETE — AP1 factory reset, onboarded, firmware updated, PPSK internet confirmed
- Policy updated to KB_School_AP_VLAN10_VLAN1: Staff → VLAN1, Students → VLAN10
- User group assignment rules: KB_School_Staff → VLAN1 (untagged/Default), KB_School_Students → VLAN10 (tagged)

## Policy: KB_School_AP_VLAN10_VLAN1 (Final — Karl's VIQ)
- SSID: KB_School_Broadcast
- Auth: PPSK, WPA2, CCMP
- Staff_PPSK → VLAN1 (Default) → untagged on port 3 → 192.168.0.x from QF-Modem
- Students_PPSK → VLAN10 → tagged on port 3 → 10.10.0.x from SW1 DHCP
- No SW1 CLI changes needed — port 3 tagging already correct (Default untagged, VLAN10 tagged)

## Key Lesson — VLAN1 for Staff
When XIQ assigns a client to VLAN1 (Default), the AP sends frames UNTAGGED.
Port 3 native VLAN = Default (VLAN1) already handles untagged frames correctly.
No switch config change needed to support staff on VLAN1.
- QF-Modem static route 10.10.0.0/24 → 192.168.0.28: ✅ (confirmed present)
- enable ipforwarding vlan Default: ✅ CRITICAL — missing after factory reset, fixed manually + saved

## Critical Fix (May 14 — after PPSK test)
`enable ipforwarding vlan Default` was missing. SW1 had the correct routing table
(default route via 192.168.0.1 BOOTP-learned) but could not forward VLAN10 client
traffic out through Default VLAN to QF-Modem without ipforwarding enabled on Default.
This must be run after EVERY factory reset — XIQ does NOT push this for Default VLAN.

## XIQ Switch Template Fix Needed (Tomorrow)
The XIQ template still has VLAN10 as untagged on port 3 (the wrong config). Need to update:
- Port 3 Port Type: Default VLAN native (untagged) + VLAN10 tagged
This ensures XIQ is the authoritative source and future deploys are correct.

## PPSK VLAN Split Verification (May 15 — CONFIRMED ✅)
Staff passphrase → MacBook 192.168.0.10 on Default VLAN ✅
Students passphrase → two clients 10.10.0.10 + 10.10.0.11 on VLAN10 ✅

### show fdb ports 3 (verbatim)
```
00:e6:0e:55:66:80   Default(0001)   AP1 management — correct native VLAN ✅
74:a6:cd:8b:19:60   VLAN10(0010)    Students client (MacBook, switched from Staff test)
76:f6:58:7c:18:00   VLAN10(0010)    Students client (iPhone)
```

### show iparp vlan VLAN10
```
10.10.0.10   76:f6:58:7c:18:00   (iPhone)
10.10.0.11   74:a6:cd:8b:19:60   (MacBook — Students passphrase)
```

### show iparp vlan Default
```
192.168.0.10   74:a6:cd:8b:19:60   (MacBook — stale from Staff passphrase test, age=20 → expiring)
192.168.0.1    58:13:d3:9c:c1:ee   (QF-Modem gateway)
```

### Dual-MAC Observation (forensic proof)
MAC 74:a6:cd:8b:19:60 (MacBook) appears in BOTH ARP tables:
- Default ARP: 192.168.0.10 at age=20 (near expiry) — from Staff passphrase test
- VLAN10 ARP: 10.10.0.11 — from Students passphrase test (current active state)
FDB shows MAC currently under VLAN10 only → confirms PPSK reassignment worked.

### Supplemental CLI Note (May 15 clarification)
`enable ipforwarding vlan Default` has the SAME idempotency problem as `enable dhcp ports 3`.
Do NOT add it to Supplemental CLI template. Keep as manual post-factory-reset step.
3-pass Supplemental CLI method still applies to ipforwarding if needed on fresh switch.

### IQAgent Heartbeat (confirmed XIQ connected)
```
INFO[2026/05/15 17:04:07] proxy device-connector unknown POST /device-connect/rest/v1/health-check/FJ012544G-00233
```
FJ012544G-00233 = SW1 serial. "unknown" is a proxy log level — not an error. SW1 → XIQ cloud ✅
