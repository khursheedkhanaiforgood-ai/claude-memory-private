---
name: XIQ Supplemental CLI — VLAN Name Gotchas
description: Supplemental CLI ${vlan:} variable resolves to VLAN ID not name. Always hardcode VLAN name. Check exact name with show vlan before writing CLI.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
Never use `${vlan:VLAN_OBJECT_NAME}` in XIQ Supplemental CLI for EXOS switches.

**Why:** The variable resolves to the VLAN ID number (e.g. `100`), not the VLAN name. EXOS `configure vlan` requires the name, not the ID. Error: `%% A number within the range of 1-4094 is expected`.

**How to apply:**
1. After XIQ pushes the switch template, SSH to switch and run `show vlan` to see the exact VLAN name XIQ created
2. Hardcode that name in the Supplemental CLI commands
3. XIQ VLAN naming convention: drops underscores (e.g. XIQ object `Guest_100` → switch VLAN name `Guest100`)
4. Double-check with: `show config | i <keyword>` before writing CLI

**Verified working syntax (VLAN 100 DHCP on SW1):**
```
configure vlan Guest100 dhcp-address-range 10.100.0.10 - 10.100.0.254
configure vlan Guest100 dhcp-options default-gateway 10.100.0.1
configure vlan Guest100 dhcp-options dns-server primary 8.8.8.8
configure vlan Guest100 dhcp-options dns-server secondary 1.1.1.1
configure vlan Guest100 dhcp-lease-timer 86400
enable dhcp ports 3 vlan Guest100
```
