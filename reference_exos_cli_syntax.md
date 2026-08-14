---
name: EXOS SwitchEngine CLI Syntax Reference
description: Verified correct EXOS SwitchEngine 33.x CLI syntax for DHCP server, routing, port config — confirmed working on 5320-16P April 7 2026. Use this exactly, no variations.
type: reference
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## DHCP Server — Complete Working Sequence (verified April 7 2026)

```
configure vlan <vlan_name> dhcp-address-range <start_ip> - <end_ip>
configure vlan <vlan_name> dhcp-options default-gateway <svi_ip>
configure vlan <vlan_name> dhcp-options dns-server primary 8.8.8.8
configure vlan <vlan_name> dhcp-options dns-server secondary 8.8.4.4
enable dhcp vlan <vlan_name>
save configuration
```

⚠️ **CRITICAL DISTINCTION — confirmed Aug 13 2026:**
- `enable dhcp vlan <name>` = enables DHCP **SERVER** on that VLAN ← use this
- `enable dhcp ports <port> vlan <name>` = enables DHCP **CLIENT** on that port (switch requests IP from upstream) ← NEVER use for server role

### Real example (SW1, VLANs 20 + 30, AP on port 3):
```
configure vlan SW_VLAN_20 dhcp-address-range 10.20.0.100 - 10.20.0.200
configure vlan SW_VLAN_20 dhcp-options default-gateway 10.20.0.1
configure vlan SW_VLAN_20 dhcp-options dns-server primary 8.8.8.8
configure vlan SW_VLAN_20 dhcp-options dns-server secondary 8.8.4.4
enable dhcp vlan SW_VLAN_20

configure vlan SW_VLAN_30 dhcp-address-range 10.30.0.100 - 10.30.0.200
configure vlan SW_VLAN_30 dhcp-options default-gateway 10.30.0.1
configure vlan SW_VLAN_30 dhcp-options dns-server primary 8.8.8.8
configure vlan SW_VLAN_30 dhcp-options dns-server secondary 8.8.4.4
enable dhcp vlan SW_VLAN_30
save configuration
```

### Real example (SW2, VLANs 50 + 60, AP on port 3):
```
configure vlan SW_VLAN_50 dhcp-address-range 10.50.0.100 - 10.50.0.200
configure vlan SW_VLAN_50 dhcp-options default-gateway 10.50.0.1
configure vlan SW_VLAN_50 dhcp-options dns-server primary 8.8.8.8
configure vlan SW_VLAN_50 dhcp-options dns-server secondary 8.8.4.4
enable dhcp ports 3 vlan SW_VLAN_50

configure vlan SW_VLAN_60 dhcp-address-range 10.60.0.100 - 10.60.0.200
configure vlan SW_VLAN_60 dhcp-options default-gateway 10.60.0.1
configure vlan SW_VLAN_60 dhcp-options dns-server primary 8.8.8.8
configure vlan SW_VLAN_60 dhcp-options dns-server secondary 8.8.4.4
enable dhcp ports 3 vlan SW_VLAN_60
save configuration
```

---

## Default Route + IP Forwarding (BOTH required for internet)
```
configure iproute add default <gateway_ip>
enable ipforwarding vlan Default
save configuration
```
Example: `configure iproute add default 192.168.0.1`

⚠ **CRITICAL:** `enable ipforwarding vlan Default` is MANDATORY after every factory reset.
Return traffic from the internet arrives on VLAN 1 (Default — the uplink to Quantum Fiber).
Without this, clients get DHCP + gateway but Safari times out. Confirmed March 29, April 7, and May 14 2026.
XIQ does NOT push this command — must be applied manually or via Supplemental CLI.

⚠ **DO NOT manually add default route** — EXOS installs it automatically from the DHCP lease
(origin `bo` in `show iproute`). Only `enable ipforwarding vlan Default` is manual.

---

## Verify
```
show dhcp-server
show iproute
show vlan
```

`show dhcp-server` must show:
- Address Range populated (e.g. 10.20.0.100->10.20.0.200)
- Ports DHCP Enabled: 3 (not "No ports enabled")
- DNS servers present
- (Gateway does not appear in show dhcp-server output — that is normal)

---

## VLAN Naming Rules
- `Mgmt` and `Management` are reserved keywords in EXOS — use `SwMgmt` or `APMgmt`
- VLAN 1 (Default) = XIQ management lifeline — never add to XIQ Routing section

---

## Config Backup — Pre-XIQ-Deploy Procedure (confirmed May 13 2026)

### On SW1 — save named backup
```
save config backup_YYYYMMDD_pre_xiq
# Prompts:
#   Save? → Yes
#   Make default? → NO  ← critical, keep primary.cfg as boot default
```

### If you accidentally say Yes to "make default":
```
use configuration primary
# Restores primary.cfg as boot + save target immediately
```

### Pull off-switch copy to Mac
```bash
# macOS requires -oHostKeyAlgorithms=+ssh-rsa (ssh-rsa deprecated in OpenSSH 8.8+)
ssh -oHostKeyAlgorithms=+ssh-rsa admin@192.168.0.28 "show configuration" > ~/Desktop/sw1_backup_YYYYMMDD.txt
wc -l ~/Desktop/sw1_backup_YYYYMMDD.txt   # verify: should be 300+ lines
```

### Revert if needed
```
# On SW1
use configuration backup_YYYYMMDD_pre_xiq
reboot
```

### Verify correct default after backup
```
show switch | include "Config"   # must show primary.cfg
```
