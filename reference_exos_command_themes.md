---
name: EXOS SwitchEngine — Thematic Command Index (4 Lenses)
description: EXOS commands organized into Design / Configure / Optimize / Troubleshoot lenses, mirroring AP3000 + VOSS structure. Source: SW1 EXOS tech-support May 7. Pair w/ reference_exos_cli_syntax.md.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Source / Context
- Platform: Extreme 5320-16P-2MXT-2X-SwitchEngine (EXOS 33.x)
- Reference dump: `docs/data/may7-dhcp/tech_support_SW1_EXOS_May7_2026.txt` (1.07 MB, 2026-05-07 10:08 EDT)
- CLI prompt: `5320-16P-2MXT-2X-SwitchEngine.X #`
- Pair: `reference_exos_cli_syntax.md` (verified syntax) + `reference_exos_factory_reset.md` (reset gotcha)

## How to use
4 lenses match the same structure as AP3000 (`reference_ap3000_command_themes.md`) and VOSS (`reference_voss_command_themes.md`). Pick the lens for your task.

| Lens | Core question |
|---|---|
| 🎨 **DESIGN** | "What's the topology, capacity envelope, redundancy story?" |
| 🔧 **CONFIGURE** | "What VLANs, ports, ACLs, services do users need?" |
| 📈 **OPTIMIZE** | "Where's the bottleneck — port utilization, queue, FDB pressure?" |
| 🩺 **TROUBLESHOOT** | "Where did the frame die — port, VLAN, FDB, ARP, route?" |

---

## 🎨 DESIGN — topology, capacity, hardware envelope

```
show switch                          # serial, model, mgmt IP, uptime
show version detail                  # OS version, image set
show ports info detail               # all ports — admin, oper, speed, duplex, MDI
show ports configuration             # configured port settings
show vlan                            # VLAN table — all VLANs, ports, IPs, flags
show stpd                            # Spanning Tree domains
show stpd s0 ports                   # STP per port (root/desg/block)
show iproute                         # routing table
show ipconfig                        # SVI / management IP config
show lacp                            # link aggregation groups
show vrrp                            # VRRP virtual router state (if used)
show qosprofile                      # QoS profile assignments
show capacity-planning               # FDB, ARP, route, ACL capacity vs limits
show power                           # PoE budget (5320 = PoE+ switch)
show power detail
show fans / show temperature         # environmentals
```

---

## 🔧 CONFIGURE — services, VLANs, ports, security

### Running config & inventory
```
show configuration                   # running config (full)
show configuration vlan              # just VLAN section
show configuration ports             # just port section
show configuration vrrp / acl / dhcp / mgmt
```

### VLANs & port membership
```
show vlan                            # quick VLAN status
show vlan tag <tag-id>
show vlan <name> ports               # which ports (tagged/untagged) belong
configure vlan <name> add ports <list> tagged|untagged
configure vlan <name> ipaddress <ip>/<mask>
enable ipforwarding vlan <name>      # ← KARL RULE — without this, no L3 routing
```

### DHCP server
```
show dhcp-server                     # all VLANs, pools, port-bindings, leases
show dhcp-server <vlan> leases
configure vlan <vlan> dhcp-address-range <start> - <end>
configure vlan <vlan> dhcp-options default-gateway <ip>
configure vlan <vlan> dhcp-options dns-server <ip>
enable dhcp ports <port-list> vlan <vlan>    # ← MUST include AP-facing port
```

### LLDP / EDP (link discovery)
```
show lldp neighbors
show lldp neighbors detail
show edp neighbors                   # Extreme Discovery Protocol (legacy)
show edp ports
```

### ACLs / Policies (EXOS uses POLICY syntax for L3 ACLs)
```
show policy                          # all policies
show policy <name>
edit policy <name>                   # vi-style editor (Esc:wq to save)
show access-list                     # ACL on ports/VLANs
show access-list configuration ports <list>
```

### Routing & static routes
```
show iproute
show iproute static
configure iproute add <dest>/<mask> <gw>
```

### Authentication / Network Login (NetLogin)
```
show netlogin
show netlogin port <num>
show netlogin all                    # full table
show radius
show tacacs
```

---

## 📈 OPTIMIZE — performance, counters, capacity

### Port utilization & errors
```
show ports utilization                # live %  per port
show ports utilization rxpkts        # RX packet rate
show ports utilization txpkts        # TX packet rate
show ports statistics                # full counter set per port
show ports statistics no-refresh
show ports collisions                # collision counters
show ports congestion
show ports rxerrors                  # RX errors (CRC, frame, undersize)
show ports txerrors                  # TX errors (deferred, late-coll)
show ports info detail               # per-port everything
show ports debounce                  # link debounce timers
clear counters ports <list>          # baseline before measuring
```

### FDB / MAC table pressure
```
show fdb                             # full FDB
show fdb stats                       # entry counts, additions, deletions
show fdb vlan <name>                 # MACs per VLAN
show fdb port <num>                  # MACs per port
show fdb mac <mac>                   # find a MAC
show fdb aging                       # FDB aging timer
configure fdb agingtime <seconds>
```

### ARP table
```
show iparp                           # full ARP
show iparp vlan <name>
show iparp <ip>
configure iparp timeout <minutes>
```

### Multicast (IGMP snooping)
```
show igmp
show igmp snooping
show igmp snooping vlan <name>
show l2stats
```

### CPU / process
```
show process
show cpu-monitoring
show memory
show memory detail
```

### Logging & event analysis
```
show log                             # syslog
show log filter <name>
show log warning
show log severity <level>
show log target                      # log destinations (syslog server etc.)
show log filter event-conditions
```

---

## 🩺 TROUBLESHOOT — where the frame died

### Stage 1 — link / physical
```
show ports info port-number          # port admin/oper/speed
show ports configuration port-number
show ports rxerrors port-number      # CRC = bad cable / SFP
show ports txerrors port-number
show ports debounce
show port <num> transceiver information detail   # SFP/SFP+ DDM
```

### Stage 2 — L2 / VLAN / FDB / STP
```
show fdb port <num>                  # which MACs learned on this port
show fdb vlan <name>                 # which MACs in this VLAN
show vlan <name>                     # VLAN flags (forwarding f, ipfwd, ifsa)
show stpd s0 ports <num>             # STP state — block? forward?
show stpd detail
show edp ports <num>                 # is the AP / switch identifying itself?
show lldp neighbors port <num>
```

### Stage 3 — L3 / ARP / routing
```
show iparp vlan <name>               # ARP — is the MAC mapped to an IP?
show iproute                         # routing table
show iproute reserved-entries        # hardware route cache
show ipstats                         # IP layer counters
configure iparp add <ip> <mac>       # static ARP if needed for testing
clear iparp                          # if you suspect stale ARP
```

### Stage 4 — DHCP server
```
show dhcp-server                     # is the AP-facing port DHCP-enabled?
show dhcp-server <vlan> leases       # is the client's MAC there?
show log filter "DHCP"               # DHCP events in syslog
clear dhcp-server <vlan> bindings    # force client to renew
```

### Stage 5 — packet capture / mirroring
```
show mirroring                       # current mirror config
configure mirroring add port <src> port <dst>
                                     # then connect Wireshark to mirror dst port
enable mirroring to port <dst>
disable mirroring
```

### CLI debug & live capture
```
debug             # (in privileged mode — context-specific debug)
show log filter "INTERESTING_KEYWORD"
show snmp                            # SNMP state for external monitoring
```

### Stress / sweep
```
ping <ip> count <num>
ping vr VR-Default <ip>              # ping in a specific VR
traceroute <ip>
```

---

## ⚠️ Operational guardrails

| Guardrail | Why |
|---|---|
| `save configuration` after any change | EXOS does NOT auto-save; reboot loses unsaved config |
| `clear counters ports` BEFORE measuring | Prevents accumulating-since-boot numbers from skewing analysis |
| `unconfigure switch all` is the **factory reset** | NOT `delete /intflash/config.cfg` (that's VOSS Fabric Engine — see `feedback_exos_factory_reset.md`) |
| `show capacity-planning` BEFORE adding 1000s of FDB entries | 5320-16P has finite TCAM/FDB; check headroom |
| `Karl Rule`: `enable ipforwarding vlan Default` | Without this, clients DHCP fine but can't reach the internet |

---

## Mapping to the Symptom→Hypothesis→Test→Validation framework

| Step | Lens | EXOS commands you'd reach for |
|---|---|---|
| Symptom | Troubleshoot | `show port info`, `show fdb port`, `show iparp vlan`, client capture |
| Hypothesis | (mental work) | use the 4-table framework on the AP side, then check switch FDB/ARP for evidence |
| Test | Troubleshoot | `show fdb vlan`, `show dhcp-server leases`, `show log filter`, mirror→Wireshark |
| Validation | Optimize | `show ports utilization` t0 vs t30 diff; `show fdb stats` for MAC churn |

---

## Cross-references
- `reference_exos_cli_syntax.md` — verified syntax (DHCP, routing gotchas)
- `feedback_exos_factory_reset.md` — `unconfigure switch all` is the right reset
- `reference_dhcp_wifi_triage_runbook.md` — 5-stage triage runbook (this index supports stages 3, 4)
- `docs/data/may7-dhcp/tech_support_SW1_EXOS_May7_2026.txt` — full live tech-support dump for reference