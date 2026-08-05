---
name: 5320 E2E Deployment Session — April 8 2026
description: Morning session — timed 30min E2E walkthrough of 2-switch 2-AP deployment via XIQ + CLI. Real-time guided exercise.
type: project
---

## Session Goal
Timed 30-minute E2E deployment of 2x 5320 + 2x AP3000 via XIQ + CLI.
Key lesson from yesterday: XIQ handles L2, CLI handles DHCP/DNS/ipforwarding.

## Start Time
April 8 2026 — morning

## E2E Sequence (reference)
1. Factory reset → ZTP+ → XIQ onboard
2. XIQ: switch policy + AP policies
3. XIQ: VLAN Attributes (20/30/50/60)
4. XIQ: Network Allocation (subnets)
5. XIQ: Routing (SVI per device, IPv4 Forwarding)
6. XIQ: Port Types (AP_Port trunk, native VLAN 1, tagged 20/30/50/60)
7. XIQ: Push config
8. CLI: DHCP server (dhcp-address-range, default-gateway, dns-server, enable dhcp ports 3)
9. CLI: enable ipforwarding vlan Default
10. CLI: save + verify (show dhcp-server, show iproute)
11. Test: iPhone IP + internet

## Status
In progress
