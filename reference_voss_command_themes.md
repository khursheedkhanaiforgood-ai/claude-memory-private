---
name: VOSS FabricEngine — Thematic Command Index (4 Lenses)
description: VOSS commands organized into Design / Configure / Optimize / Troubleshoot lenses, mirroring AP3000 + EXOS structure. Layered for Fabric (IS-IS, SPB, I-SID, FA). Pair w/ reference_voss_cli.md.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Source / Context
- Platform: Extreme 5320-16P-2MXT-2X-FabricEngine (VOSS 9.3.x)
- SW2 reference: `KhKLab-SW-01` (project_5320_new_arch_voss_ipe.md)
- CLI prompt: `Switch:1>` or `Switch:1#` (privileged) or `Switch:1(config)#`
- Pair: `reference_voss_cli.md` (verified syntax) + `reference_exos_voss_4_principles.md` (mental model)

## How to use
4 lenses match AP3000 + EXOS structure. Same lens, different platform — pick the lens for your task. The fabric story (IS-IS / SPB / I-SID / FA) becomes the **Design** anchor that the other lenses reinforce.

| Lens | Core question (fabric flavor) |
|---|---|
| 🎨 **DESIGN** | "Where am I in the fabric — Backbone Edge, Backbone Core, or BCB? What I-SIDs am I a member of?" |
| 🔧 **CONFIGURE** | "What service (I-SID) am I creating, and how does it map to a local VLAN?" |
| 📈 **OPTIMIZE** | "Is IS-IS converged? Are SPB tree builds clean? Is FA delivering on the edge?" |
| 🩺 **TROUBLESHOOT** | "Where did the I-SID break — adjacency, SPB tree, FA assignment, DHCP?" |

---

## 🎨 DESIGN — fabric position, identity, hardware envelope

### Identity & area
```
show sys-info                            # serial, model, OS version, uptime
show sys-id                              # this switch's IS-IS system-id
show isis                                # IS-IS state — area, type, hostname
show isis spbm                           # SPB state — Nick-name, B-VID, primary B-VID
show isis manual-area                    # configured 49.b0b1 area address
show isis interface                      # IS-IS-enabled interfaces
show isis adjacencies                    # IS-IS L1 adjacency table
```

### Topology & SPB
```
show isis lsdb                           # link-state DB (the SPB topology)
show isis lsdb detail
show isis spbm i-sid                     # all I-SIDs this switch knows
show isis spbm i-sid all                 # full I-SID table
show isis spbm unicast-fib               # unicast forwarding info base
show isis spbm multicast-fib             # multicast FIB
show isis spbm tree                      # SPB tree per B-VID
show ip arp vrf GlobalRouter             # ARP table on the global VRF
```

### Platform / hardware
```
show sys hw-info
show platform port                       # 1/1, 1/2, ... mapping
show ports
show interface gigabitEthernet 1/<n>     # per-port detail
show poe                                 # PoE state (5320 has PoE+)
show fan
show temperature
show running-config                      # full config dump
show config-version
```

---

## 🔧 CONFIGURE — services (I-SIDs), VLANs, ports, DHCP

### Service creation flow (the VOSS way)
```
configure terminal
router isis
  manual-area 49.b0b1
  spbm 1 nick-name 0.00.02              # ← UNIQUE per switch in fabric
exit
router isis enable                       # ← enable AFTER nick-name set

vlan create 70 name "Corp_New" type port-mstprstp 0
vlan i-sid 70 100070                     # ← I-SID = VLAN + 100,000 convention
```

### VLANs / I-SIDs
```
show vlan                                # all VLANs
show vlan i-sid                          # VLAN ↔ I-SID mappings
show vlan members                        # port membership
show vlan members <vid>
show vlan basic
```

### DHCP server (ON SW2 itself, not relay)
```
ip dhcp-server subnet 10.70.0.0/24
  pool 10.70.0.100 10.70.0.200
  router 10.70.0.1
  domain-name-servers 8.8.8.8
exit
ip dhcp-server enable

show ip dhcp-server                      # DHCP server config
show ip dhcp-server lease                # active leases
show ip dhcp-server statistics
```

**VOSS DHCP keyword cheatsheet (NOT EXOS!):**
- `subnet` (not `pool`)
- `pool <start> <end>` (the IP range)
- `router` (not `default-router`)
- `domain-name-servers` (not `dns-servers`)

### Fabric Attach (the auto-provisioning edge)
```
interface gigabitEthernet 1/3
  auto-sense enable
  fa enable
  no shutdown
exit

show fa assignment                       # active FA bindings (port → I-SID)
show fa elements                         # FA-capable neighbors detected
show fa interface                        # FA state per interface
show fa i-sid                            # I-SIDs offered via FA
```

### Routes
```
ip route 0.0.0.0/0 <next-hop> weight 1   # create
ip route 0.0.0.0/0 <next-hop> enable     # activate (separate command!)
show ip route                            # full route table
show ip route static
show ip route summary
ip-shortcut enable                       # under `router isis` — makes this an exit point
```

### Management
```
sys name <hostname>                      # set hostname
ip name-server primary <ip>              # ⚠ does NOT feed IQAgent DNS — must use DHCP at boot
                                         # (see feedback_voss_iqagent_dns.md)
show ip name-server
show clip                                # circuitless IP (loopback) for management
clip 1 vrf GlobalRouter ip 10.158.4.1/32
```

---

## 📈 OPTIMIZE — fabric health, port stats, traffic flow

### IS-IS / SPB convergence
```
show isis statistics                     # PDU stats
show isis adjacencies detail             # adjacency state, hold time, PDU counts
show isis interface counters             # per-interface PDU counts
show isis spbm interface                 # SPB per-interface state
show isis spbm metric
show ip route protocol isis              # routes learned via IS-IS
```

### Port statistics
```
show interface gigabitEthernet 1/<n> statistics
show port statistics                     # summary
show port-statistics                     # alternate
show port-statistics <port>              # specific port
clear port-statistics <port>             # baseline before measuring
show interface gigabitEthernet rate-limit
```

### Multicast
```
show ip igmp interface
show ip igmp snooping
show ip igmp groups
show isis spbm multicast-fib             # SPB multicast tree
```

### MAC / FDB
```
show vlan mac-address-table
show vlan mac-address-table vlan <vid>
show vlan mac-address-table interface gigabitEthernet 1/<n>
show vlan fdb-static
show vlan fdb-aging
```

### CPU / memory
```
show sys-info utilization
show memory-utilization
show top                                 # process viewer
```

### Telemetry / cloud agents
```
show iqagent                             # XIQ cloud agent state
show iqagent connection                  # cloud connection
show iqagent log                         # IQAgent log
```

### Logging
```
show log file                            # log file content
show log file tail
show log
```

---

## 🩺 TROUBLESHOOT — where the I-SID broke

### Stage 1 — fabric adjacency
```
show isis adjacencies                    # are adjacencies UP?
show isis adjacencies detail
show isis statistics                     # any PDU errors?
show isis interface
show isis interface ifindex <n>
```

### Stage 2 — SPB tree / I-SID propagation
```
show isis spbm i-sid all                 # is the I-SID known here?
show isis spbm unicast-fib c-mac <mac>   # which fabric path leads to this MAC?
show isis spbm tree
show isis lsdb
show isis lsdb detail level 1
```

### Stage 3 — Fabric Attach (port-level provisioning)
```
show fa assignment                       # is the AP's port showing dynamic I-SIDs?
show fa elements                         # is the AP being detected?
show fa interface gigabitEthernet 1/3
show fa i-sid                            # what I-SIDs are being advertised
```

### Stage 4 — local VLAN ↔ I-SID binding
```
show vlan i-sid                          # local VLAN/I-SID mapping correct?
show vlan members <vid>                  # is the right port a member?
show ip dhcp-server lease                # did the client get a lease?
show ip arp                              # ARP for the client present?
```

### Stage 5 — packet capture / mirroring
```
mirror by-port <src-port> destination <dst-port>     # set up port mirror
show mirror
no mirror by-port <src-port>             # disable
```

### CPU / process / health
```
show sys-info
show sys-info utilization
show log file tail                       # recent events
show clock
```

### Diagnostic ping / traceroute
```
ping <ip>
ping <ip> source <source-ip>
traceroute <ip>
ping isis l-isis-id <sys-id>             # IS-IS-level ping (fabric ping)
traceroute isis                          # SPB traceroute
```

---

## ⚠️ Operational guardrails

| Guardrail | Why |
|---|---|
| `save config` after any change | VOSS does NOT auto-save (same as EXOS). Reboot loses unsaved config. |
| `no router isis enable` BEFORE changing fabric IDs | Runtime fabric ID changes are blocked. Disable IS-IS, change, re-enable. |
| Ensure NICK-NAME is **unique per switch** in fabric | Duplicate nick-names break SPB tree builds — silent forwarding loops/blackholes. |
| **Do NOT use `ip name-server` for IQAgent DNS** | IQAgent reads `/etc/resolv.conf` populated only at DHCP boot. See `feedback_voss_iqagent_dns.md`. |
| Factory reset = `delete /intflash/config.cfg` + `reload` | (NOT `unconfigure switch all` — that's EXOS). |
| `auto-sense` on a port managed by FA = managed automatically | Don't manually configure VLANs/ACLs on FA-managed ports — they conflict. |

---

## Mapping to the 4 Principles (EXOS→VOSS Rosetta Stone)

| Principle | Lens command pattern |
|---|---|
| 1. Brain (IS-IS) | DESIGN: `show isis adjacencies`, `show isis lsdb` · TROUBLESHOOT: `show isis statistics` |
| 2. Muscle (SPB) | DESIGN: `show isis spbm tree` · TROUBLESHOOT: `show isis spbm unicast-fib` |
| 3. Service (I-SID) | CONFIGURE: `vlan i-sid X 100X` · TROUBLESHOOT: `show isis spbm i-sid` |
| 4. Edge (FA / Auto-Sense) | CONFIGURE: `auto-sense enable` + `fa enable` · TROUBLESHOOT: `show fa assignment` |

## Mapping to the Symptom→Hypothesis→Test→Validation framework

| Step | Lens | VOSS commands |
|---|---|---|
| Symptom | Troubleshoot | `show fa assignment`, `show isis adjacencies`, `show ip dhcp-server lease` |
| Hypothesis | (mental) | which of 4 principles is broken? Brain / Muscle / Service / Edge |
| Test | Troubleshoot | `show isis lsdb`, `show isis spbm i-sid`, mirror→Wireshark on edge port |
| Validation | Optimize | `show port-statistics` t0 vs t30 diff; FA assignment table check |

---

## Cross-references
- `reference_voss_cli.md` — verified VOSS CLI syntax (DHCP, ISIS, VLAN, FA)
- `reference_exos_voss_4_principles.md` — Brain/Muscle/Service/Edge mental model
- `feedback_voss_iqagent_dns.md` — DHCP-at-boot requirement for IQAgent DNS
- `project_5320_new_arch_voss_ipe.md` — SW2 deployment context (Corp_New VLAN 70/I-SID 100070, Guest_New VLAN 80/I-SID 100080, Port 1/1=IPE, 1/3=AP2, 1/5=MacBook)