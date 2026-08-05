---
name: AP3000 (IQ Engine OS) — Thematic Command Index
description: Commands organized for Socratic mode across 4 lenses (Design / Configure / Optimize / Troubleshoot). Includes show / debug / kdebug / capture / filter. Source: AP1 SSH dump May 7 2026 + AP3000 tech-support. Pair w/ reference_dhcp_wifi_triage_runbook.md.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Source files
- AP1 SSH dump (verbose CLI menu): `/Users/khukhan/Downloads/AP1_SSH_Dump_May 7 2026_1720.docx`
- AP1 tech-support (running state): `/Users/khukhan/5320-onboarding-agent/docs/data/may7-dhcp/tech_support_AP1_May7_2026.txt`
- SW1 EXOS tech-support (cross-layer): `/Users/khukhan/5320-onboarding-agent/docs/data/may7-dhcp/tech_support_SW1_EXOS_May7_2026.txt`
- AP CLI prompt: `AH-556680#` (model AP3000, IQ Engine OS / formerly HiveOS)

## How to use this file
**4 lenses** map to the 4 phases of any AP work. Pick the lens for the current task; the commands below are organized so you don't have to memorize the whole CLI tree — just the lens entry-point.

| Lens | When | Core question |
|---|---|---|
| 🎨 **DESIGN** | Before any deployment | "What does the radio environment require?" |
| 🔧 **CONFIGURE** | Initial bring-up | "What services, security, VLANs, SSIDs do clients need?" |
| 📈 **OPTIMIZE** | Steady-state tuning | "Where is airtime / throughput / coverage suboptimal?" |
| 🩺 **TROUBLESHOOT** | Active failure | "What broke, where, and how do I prove it?" |

Underscore-prefixed commands (`_show`, `_debug`, `_kdebug`) are hidden / low-level / use-with-care. Public versions exist for most of them.

---

## 🎨 DESIGN — RF environment, capacity, channel planning

### Radio characteristics
```
show radio profile [ <name> ]            # 11ax-2g / 11ax-5g / 11ax-6g profiles
show interface <wifix> channel           # current channel + width
show interface <wifix> _per-chain        # per-spatial-stream RSSI
show interface <wifix> sdr               # software-defined-radio snapshots
show interface <wifix> dfs [ _detail ]   # DFS state + radar history
show interface <wifix> _spectral         # spectral analysis
```

### ACSP (channel planning) + AI-RRM
```
show acsp                                # Advanced Channel Selection Protocol
show acsp _detail
show ai-rrm                              # AI Radio Resource Mgmt (cloud-driven)
show acsp <wifix>
```

### Coverage, neighbors, mesh
```
show amrp                                # Advanced Mobility Routing Protocol
show nbr                                 # neighbor table (other APs)
show mesh                                # mesh fabric (if enabled)
show interference                        # interference map (CU + CRC thresholds)
```

### LLDP / CDP (upstream switch visibility)
```
show lldp                                # what does the AP see on its eth port
show cdp                                 # Cisco neighbor discovery (if enabled)
```

### Capacity guardrails (for design SLAs)
```
show capture local _free_space           # local capture buffer headroom
show system temperature [ _detail ]      # thermal envelope
```

---

## 🔧 CONFIGURE — services, security, SSIDs, VLANs

### Running config (start here)
```
show config running                      # full running configuration
show running-config [ _all ]             # alternate form
show running-config password             # cleartext PSK / cred view (privileged)
show running-config users [ password ] [ all ]
show config version                      # config revision counter
```

### SSID & security
```
show ssid [ <name> ]                     # SSID list + state
show ssid <name> security wlan dos       # DoS protection on this SSID
show ssid <name> security screening [ detail ]
show ssid <name> manage
show ssid <name> schedule [ detail ]
show ssid-schedule
show auth [ interface <wifix.y|ethx> ]   # auth config per interface
show auth private-psk                    # PPSK / per-user PSK
show auth mac-binding <ssid> [ <mac> ]   # MAC-locked PSK bindings
```

### User-profile / role-based policy
```
show user-profile-policy [ <name> ]      # per-user-profile policies
show user-group [ <name> ]               # group definitions
show user [ _key-id ]                    # users
show ssid <name> user-group              # which groups attach to which SSID
show mobile-device-policy [ <name> ]
```

### IP-object / device-group / firewall objects (XIQ Common Objects mirror)
```
show mac-object [ <name> ]               # MAC-based ACL objects
show os-object [ <name> ]                # OS-fingerprint objects
show domain-object [ <name> ]            # FQDN ACL objects
show device-group [ <name> ]
show os-detection                        # OS-fingerprinting status
```

### Hotspot / Passpoint / Captive Portal
```
show hotspot profile [ <name> ]
show hotspot osu-provider [ <name> ]
```

### QoS (classification + marking + policy)
```
show qos                                 # QoS overview
show qos classifier-profile [ <name> ]
show qos classifier-map 8021p [ <num> ]  # CoS classification
show qos classifier-map 80211e [ <num> ] # WMM classification
show qos classifier-map diffserv [ <num> ]
show qos classifier-map ssid <name>
show qos classifier-map oui [ <oui> ]    # device-OUI-based classification
show qos classifier-map service [ <name> ]
show qos marker-profile [ <name> ]
show qos marker-map ...                  # parallel marker tree
show qos policy [ <name> ]
```

### Backhaul / Telegraf / cloud agents
```
show capwap                              # CAPWAP-style cloud backhaul state
show telegraf                            # telegraf telemetry agents
```

---

## 📈 OPTIMIZE — measure, then tune

### Per-station / per-client (the Wi-Fi unit of analysis)
```
show station [ <mac> ]                   # all associated stations + RSSI/MCS
show station <mac> counter               # per-station TX/RX counters
show station ipv6                        # IPv6-bound stations
show station mu-mimo summary             # MU-MIMO grouping
show station mu-mimo candidates
show interface mu-mimo counters
```

### Per-radio counters
```
show interface <wifix|wifix.y> counter   # frame counts, errors, retries
show interface <wifix.y> qos-classifier  # active classifications on this VAP
show interface <wifix.y> qos-marker
```

### QoS counters (per-user / per-profile / per-class)
```
show qos counter user [ <mac> ]          # this is where you see real airtime
show qos counter user-profile [ <name> ]
show qos counter ssid [ <name> ]
```

### Roaming / mobility
```
show roaming cache mac <mac>             # 802.11k/v neighbor cache
show roaming cache
show 11k                                 # neighbor reports
show 11v                                 # BSS transition mgmt
show client-load-balance status
show band-steering
```

### Application visibility
```
show application reporting               # L7 reporting state
show application
show application-essentials
show l7
```

### Multicast / IGMP / mDNS (the airtime killers)
```
show ssid <name> multicast
show interface <wifix.y> multicast [ _detail ]
show bonjour-gateway                     # Bonjour / mDNS proxy
show igmp
```

---

## 🩺 TROUBLESHOOT — the lens you'll use most

### Live state — clients & associations
```
show station [ <mac> ]                   # is the client even there?
show client-info-collection [ ip <ip> ]  # full client picture
show _client detail information [ {add-debug} ]
show 802.1x-mac-table                    # 802.1X auth table on Ethernet ports
show alg sip calls                       # voice calls in flight (if SIP ALG on)
```

### DHCP-specific (THE focus area for May 6 incident)
```
_show auth dhcp-fp-station               # DHCP-fingerprinted stations
_debug auth dhcp_fp                      # debug DHCP fingerprinting
_debug dhcpd {all|basic|relay|error|dump|verbose}   # AP-side DHCP daemon (if AP IS the server)
_debug dhclient {all}                    # AP's own DHCP client (uplink)
```

### Auth / EAPOL / WPA — the WHY behind silent decrypt failures
```
show auth [ interface <wifix.y|ethx> ]
_debug auth {error|basic|info|verbose|dump|excessive|comm|packet|sync|probe|dhcp_fp|fsm|all|...}
_kdebug dot1x {basic|error|timer|all}    # kernel-level 802.1X
_kdebug auth {basic|info|error|all}
_debug wpad {wifi|eth} {all|basic|error|info}    # WPA daemon
```

### Frame-level inspection — `kdebug wifi-driver` (the deepest you can go)
```
_kdebug wifi {basic|detail|all}
_kdebug wifi-driver <wifix.y> {input|assoc|scan}
_kdebug wifi-driver <wifix.y> extend {vlan|acl}
_kdebug wifi-driver <wifix> msglevel {error|trace|prhdrs|prpkt|info|tmp|rate|oid|assoc|prusr|ps|port|dual|dfs|cac|amsdu|ampdu|scan|tdls|pwrsel|vlan|wsec|wsec_dump|txbf|...|all}
_kdebug wifi-driver <wifix> phymsglevel {error|trace|inform|tmp|txpwr|cal|aci|radar|thermal|...}
_kdebug wifi-driver <wifix> awemsglevel {error|trace|inform|acsp|mesh|acl|bgscan|dos|ff|hdd|idp|fe|sniffer|ioctl|sensor|all}
_kdebug wifi-driver <wifix> emsglevel {error|info|scan|scan_dbg|intr|rm|ampdu|amsdu|rxtxutil|qos|probesup|event|80211raw|sas|dtim|airiq|prhdrs|assoc|ps|11kv|all}
```

**For TK / GTK / decrypt-failure tracing**: `wsec`, `wsec_dump` keys in `msglevel` ↑.

### Forwarding-engine / bridge / VLAN
```
_kdebug fe {basic|pkt|drop|forw|vlan|dev|cac|sync|station|probe|session|frag|cac_airtime|non_mesh|nat|detail|l7|ipv6|fw}
_kdebug fe_arp {basic|error|timer|all}
_kdebug mac {basic|error|info|all}       # bridge MAC table
_kdebug eth {basic|vlan|detail|dsa|all}  # eth port + VLAN tagging
_kdebug qos {basic|error|classify|police|schedule|mark|cli|debug|drop|all}
```

### System-level / hardware
```
_kdebug systop {basic|hang|hang_detail|all}
_kdebug board {basic|info|error|debug|all}
_kdebug mpi {basic|info|all}             # multi-processor interconnect
_show kdebug info                        # show what's currently being kdebug'd
_debug stop                              # stop ALL active debug
_debug show                              # what debug is on
_debug pm {basic|info|corefile|watchdog|netlink|cpu|change|all}
_debug brd {basic|info|verbose|statistics|hotplug|wanmon|...}
```

### Packet capture (built-in, no external sniffer needed)
```
capture interface <ethx|wifix> [ count <num> ] [ duration <num> ] [ size <num> ] [ filter {and|or} <filter#> ] [ promiscuous ]
capture save interface <ethx|wifix> <file>
show capture interface <ethx|wifix>
show capture local [ _free_space ]
clear capture local [ <file> ]
```

### **Frame filter syntax (THE thing you wanted to learn)**
```
filter <num> l2 [ {data|ctl|mgmt} ] [ subtype <hex> ] [ src-mac <mac> ] [ dst-mac <mac> ] [ bssid <mac> ] [ tx-mac <mac> ] [ rx-mac <mac> ] [ error {crc|decrypt|mic|all|no} ] [ etype <hex> ] [ vlan <vlan> ]
filter <num> l3 [ src-ip <ip> ] [ dst-ip <ip> ] [ protocol <num> ] [ src-port <num> ] [ dst-port <num> ]
filter <num> traffic {in|out}
filter [ <num> ] [ direction bidirectional ]
show filter [ <num> ]
```

**Key trick for DHCP debug**: build a filter for L3 udp protocol 17 src/dst port 67 or 68:
```
filter 1 l3 protocol 17 src-port 68 dst-port 67     # client → server (DISCOVER/REQUEST)
filter 2 l3 protocol 17 src-port 67 dst-port 68     # server → client (OFFER/ACK)
capture interface wifi0 duration 60 filter and 1
```

**Key trick for decrypt-failure**: filter on `error decrypt` or `error mic`:
```
filter 3 l2 error decrypt
filter 4 l2 error mic
capture interface wifi0 duration 60 filter or 3
```

### Remote sniffer (mirror to your laptop's Wireshark live)
```
exec capture remote-sniffer interface <wifix> [ user <u> <pw> ] [ host-allowed <ip> ] [ local-port <num> ] [ promiscuous ]
show capture remote-sniffer
```

### Console / logging knobs
```
debug console [ {all} ]                  # turn on console debug stream
debug console level {emergency|alert|critical|error|warning|notification|info|debug}
debug console timestamp                  # add timestamps to console
_test {log-case|log-user-case|log-flash-case} {emergency|alert|critical|error|warning|notification|info|debug}
_logging trap                            # log-trap config
```

### Specific protocol / feature debug
```
_debug rm {error|basic|info|packet|all}      # 802.11k Radio Measurement
_debug dcd {error|basic|info|chnl|power|idp|nbr|packet|...}    # DCD = Device Control Daemon
_debug scd {...|naas|radsec|...}             # SCD = Station Control Daemon
_debug capwap {...}                          # cloud backhaul protocol
_debug fed {alg_dhcp|alg_dns|alg_sip|alg_ftp|alg_tftp|...}   # ALG = App Layer Gateway
_debug ipfw {...}                            # AP firewall
_debug swd {basic|info|error|verbose|brg|fdb|vlan|event|igmp|qos|span|agg|monitor_report|client|led|ldb|cavc|acl|all}    # switching daemon (the AP's own bridge)
_debug acsd {error|scan|info|all}            # ACSP daemon
_debug sflow {...}                           # sFlow telemetry
_debug ah_telegraf {...}                     # telegraf agent
_debug nbr {...}                             # neighbor daemon
_debug bgd {...}                             # Bonjour Gateway daemon
_debug lcs {...}                             # Location services
_debug ltr {...}                             # Location-Triggered Rules
_debug vpn {...}
_debug webui {...}                           # local web UI debug
```

---

## ⚠️ Operational guardrails

| Guardrail | Why |
|---|---|
| `_debug stop` after every debug session | Debug streams to console eat CPU + flash, can OOM the AP |
| `_debug show` before walking away | See what you left running |
| `clear capture local` after pulling pcaps | Local capture buffer is small; full = capture stops silently |
| `show config version` BEFORE making changes | Note the version number; lets you compare/revert |
| Avoid `_kdebug wifi-driver msglevel all` | Floods console; can hang serial. Use specific keys (`wsec`, `assoc`, `scan`) |

---

## Mapping to the Symptom → Hypothesis → Test → Validation framework

Per `reference_dhcp_wifi_triage_runbook.md`, the 4-step framework runs across both EXOS and AP layers. AP-side commands above slot in like this:

| Framework step | Lens | Commands you'd reach for |
|---|---|---|
| **Symptom** | Troubleshoot | `show station <mac>`, `show auth interface`, AP capture w/ filter |
| **Hypothesis** | (mental work) | use the 4-table elimination from May 7 EOD |
| **Test** | Troubleshoot | `_kdebug wifi-driver msglevel wsec`, `_debug auth`, frame-filter capture |
| **Validation** | Optimize/Troubleshoot counter diff | `show counters wireless` t0 vs t30 diff |

---

## Related memory
- `reference_dhcp_wifi_triage_runbook.md` — 5-stage triage flow this command index supports
- `reference_voss_cli.md` — VOSS CLI for upstream switch correlation
- `reference_exos_cli_syntax.md` — EXOS CLI for upstream switch correlation
- `project_dhcp_macbook_incident.md` — May 6 case study where these commands matter
- `project_eapol_forensics_may4.md` — May 4 EAPOL theory (TK/GTK) that wsec_dump output validates