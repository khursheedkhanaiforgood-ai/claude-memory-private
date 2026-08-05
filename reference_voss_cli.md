---
name: VOSS / Fabric Engine CLI Reference
description: Confirmed VOSS CLI syntax for 5320 running FabricEngine 9.3.2.0. Navigation, IP, VLAN, routes, IQAgent, show commands, boot behavior, factory reset. Verified Apr 30 2026.
type: reference
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Navigation
```
enable                        → privileged exec  (#)
configure terminal            → global config    ((config)#)
application                   → app sub-mode     ((config-app)#)
interface vlan 1              → VLAN SVI         ((config-if)#)
exit                          → up one level
end                           → back to exec
```

## Hostname
```
sys name SW2-VOSS             ← in global config
```

## IP Addresses
```
interface vlan 1
  ip address 192.168.0.31 255.255.255.0
  no ip address 192.168.0.31       ← remove (no mask needed)
exit

NOTE: VOSS VLAN interface does NOT support DHCP client (ip address dhcp = invalid)
```

## Static Routes (VOSS-specific syntax)
```
ip route 0.0.0.0/0 192.168.0.1 weight 1     ← STEP 1: create (CIDR required)
ip route 0.0.0.0/0 192.168.0.1 enable        ← STEP 2: activate (separate command)

ip route ?   → shows: bfd / preference / {A.B.C.D/X} / {A.B.C.D}
ip route 0.0.0.0/0 192.168.0.1 ?  → shows: enable / local-next-hop / name / preference / weight
```

## DNS
```
ip name-server primary 8.8.8.8
ip name-server secondary 8.8.4.4

show ip dns    ← shows status (Inactive = never queried yet) + request count

WARNING: ip name-server does NOT feed IQAgent /etc/resolv.conf
         IQAgent DNS only works if DHCP ran at boot (ZTP+)
```

## VLAN Management
```
show vlan members                   ← all VLANs + port assignments
vlan members add 1 1/1              ← add port to VLAN (triggers Auto-Sense disable warning = OK)
vlan members remove 1 1/1

Default VLAN behavior: ports NOT in any VLAN by default (unlike EXOS where all ports start in VLAN 1)

VOSS auto-creates VLAN 4048 (auto-sense), 4051, 4052 (B-VLANs for SPB) on fresh boot
```

## IQAgent (XIQ Cloud)
```
configure terminal
application
  no iqagent enable                          ← stop agent first before changing server
  iqagent server hac.extremecloudiq.com      ← set XIQ server hostname
  iqagent enable                             ← start agent
  iqagent ?  → only: enable / proxy / server

WARNING: iqagent server <IP> fails — TLS cert issued for hostname not IP
WARNING: must disable before changing server or get "Operation not allowed" error
```

## Show Commands
```
show ip interface              ← all L3 interfaces + OPER status
show vlan members              ← VLAN port membership table
show ip vrf                    ← VRF table (only GlobalRouter on 5320)
show ip dns                    ← DNS server + request count
show log file | include CLOUD  ← filter log for XIQ cloud agent
show running-config | include name-server
show running-config            ← full running config
```

## SPB Nickname
```
configure terminal
no router isis enable         ← REQUIRED: disable IS-IS first (runtime change not allowed)
router isis
spbm 1 nick-name 0.00.02     ← hyphenated; inside router isis context
exit
router isis enable
exit
save config
```
Verify: `show isis spbm` → NICK NAME column shows 0.00.02
FAIL modes: (1) at exec 1> prompt → Invalid input; (2) IS-IS enabled → runtime change not allowed

## DHCP Server (confirmed syntax — May 1 2026)
```
configure terminal

ip dhcp-server subnet 10.70.0.0/24       ← enters (config-dhcp)# context
  pool 10.70.0.100 10.70.0.200           ← "pool" = address range (NOT "range")
  router 10.70.0.1                       ← "router" = default-router (NOT "default-router")
  domain-name-servers 8.8.8.8            ← correct keyword ✅
exit

ip dhcp-server enable                    ← global enable last

WRONG (NotebookLLM): ip dhcp-server pool / range / default-router — none of these exist
WRONG: interface vlan X → ip dhcp-server enable — not valid in VOSS
```

## Save / Reset
```
save config                    ← saves to /intflash/config.cfg
delete /intflash/config.cfg    ← VOSS factory reset (then reload)
reload                         ← reboot

NEVER use: unconfigure switch all (that's EXOS only)
```

## Boot Behavior — Normal Warnings (do not panic)
```
Could not open primary config file /intflash/config.cfg   ← clean boot, no config yet
ISIS Enabled without assigning the nickname               ← need to set nickname
ISIS Md5 key file does not exist                          ← normal on fresh boot
FA Md5 key file does not exist                            ← normal on fresh boot
ZTP+ discovery failed - No DNS server configured          ← if no DHCP at boot
Trial Period will expire in 24 days                       ← license trial, normal
```

## Default Login
```
Login: rwa
Password: rwa    ← forced change on first login
```

## VOSS vs EXOS Key Differences
| Topic | EXOS | VOSS |
|-------|------|------|
| Factory reset | unconfigure switch all | delete /intflash/config.cfg |
| Default port VLAN | All ports in VLAN 1 | No default — add manually |
| Static route | configure iproute add default <gw> | ip route 0.0.0.0/0 <gw> weight 1 + enable |
| Hostname | configure snmp sysName <name> | sys name <name> |
| DHCP client on VLAN | Supported | NOT supported |
| Default login | admin / (blank) | rwa / rwa |
| Save config | save configuration <name> | save config |
| Config file | /intflash/<name>.cfg | /intflash/config.cfg |

## XIQ Onboarding — Correct Sequence for VOSS
1. Delete device from XIQ inventory (if previously EXOS)
2. Connect a data port to HomeModem/DHCP source BEFORE boot
3. Boot → ZTP+ runs → DHCP gives IP + DNS → IQAgent resolves XIQ
4. Device appears in XIQ Inbox as Fabric Engine
5. Accept → assign VOSS policy → push IS-IS/SPB from XIQ
6. Static IP configured via XIQ template AFTER connection established
