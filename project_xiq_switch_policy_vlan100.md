---
name: XIQ Switch Policy — Khursheed_SW1_VLAN100
description: May 13-14 2026: Build VLAN 100 switch policy in XIQ to replace manual CLI config. SW1 factory reset done. Supplemental CLI stuck — resume in morning.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Goal
Deploy VLAN 100 (Guest100) on SW1 entirely via XIQ — no CLI. Practice for Karl's lab where EXOS CLI is unavailable.

## Why
Phase 2 of PPSK sprint: replace manual Phase 1 CLI config with XIQ-managed policy. Validates XIQ as authoritative config source.

## Policy: Khursheed_SW1_VLAN100
- Type: Switching/Routing
- Switch Template: `Khursheed_5320_For_Karl` (model: 5320-16P-2MXT-2X)

## What Was Built (May 13-14)

### Port Configuration
- Port 1 = Trunk (VLAN 1, uplink to QF-Modem)
- Port 3 = AP Port Type — VLANs 20, 30, 100 tagged (all three in Port Type)

### VLAN Object
- XIQ VLAN object name: `Guest100` (no underscore — XIQ naming convention)
- Tag: 100

### Network Allocation
- Name: `Guest_100_Net`
- Subnet: 10.100.0.0/24
- VLAN: Guest100 (100)

### Routing (SVI)
- Device: FJ012544G-00233 (5320-16P-2MXT-2X-SwitchEngine, 192.168.0.28)
- IP: 10.100.0.1/255.255.255.0
- VLAN: Guest100 (100)
- IPv4 Forwarding: ENABLED ✅

### Supplemental CLI (name: Guest_100_DHCP)
```
configure vlan Guest100 dhcp-address-range 10.100.0.10 - 10.100.0.254
configure vlan Guest100 dhcp-options default-gateway 10.100.0.1
configure vlan Guest100 dhcp-options dns-server primary 8.8.8.8
configure vlan Guest100 dhcp-options dns-server secondary 1.1.1.1
configure vlan Guest100 dhcp-lease-timer 86400
enable dhcp ports 3 vlan Guest100
```

## SW1 State After Factory Reset + XIQ Push (show vlan)
| VLAN Name   | Tag | SVI           | IP Fwd | Notes |
|-------------|-----|---------------|--------|-------|
| Default     | 1   | 192.168.0.28  | -      | QF-Modem uplink |
| Guest100    | 100 | 10.100.0.1/24 | ✅ ON  | XIQ-created |
| VLAN_0020   | 20  | none          | -      | Created by port type, no routing |
| VLAN_0030   | 30  | none          | -      | Created by port type, no routing |

VLANs 20/30 exist as empty shells (from AP Port Type) — intentional, this policy is VLAN 100 only.

## Status (end of May 14 2026)
- SW1 factory reset: ✅ DONE
- Policy created + all config built: ✅ DONE
- IQAgent: CONNECTED, polling every ~2 min
- **Supplemental CLI: ⚠️ STUCK** — XIQ config job locked, couldn't push DHCP commands
- Last Config Success Time: 06:23 UTC May 14 (pre-Supplemental CLI fix)
- No DHCP on VLAN 100 yet — `show dhcp-server` returns empty

## Resume Tomorrow Morning
1. Check if XIQ config lock has cleared (5-10 min timeout expected overnight)
2. In XIQ: Deploy Policy → Khursheed_SW1_VLAN100 → Deploy to SW1
3. Verify on SW1: `show dhcp-server` → pool 10.100.0.10-10.100.0.254, port 3 enabled
4. Connect iPhone to PPSK_Demo → verify gets 10.100.0.x → ping 8.8.8.8
5. Then tackle VLANs 20/30 routing or move to Karl's VIQ task

## Key Lessons — XIQ Supplemental CLI Gotchas
1. `${vlan:VLAN_NAME}` resolves to VLAN **ID number** (e.g. 100), NOT the VLAN name — EXOS rejects it
2. Always hardcode the VLAN name — run `show vlan` on switch first to get exact name
3. XIQ VLAN naming: drops underscores, uses its own convention (e.g. `Guest100` not `Guest_100`)
4. Check exact name BEFORE writing Supplemental CLI: `show config | i <vlan_keyword>`
