---
name: WLANPros WLPC Express 2026 Troubleshooting Framework
description: Content from WLANPros Troubleshooting - WLPC Express 2026 (1).pdf — 30-row potential causes table, 6-process troubleshooting framework, frame distribution stats. Ferney Muñoz (CWNE#187) + Keith Parsons (CWNE#3).
type: reference
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Source
`/Users/khukhan/Downloads/WLPC Troubleshooting/WLANPros Troubleshooting - WLPC Express 2026 (1).pdf`
Copyright © 2026 Wireless LAN Professionals, Inc. All Rights Reserved
Instructor: Ferney Muñoz (CWNE#187) | Director: Keith R. Parsons (CWNE#3)

---

## The 6-Process Troubleshooting Framework
Center: REQUIREMENTS (Primary Coverage, Secondary Coverage, SNR, Data Rates, Density, Capacity)

| Process | Tool/Method |
|---------|-------------|
| MANAGE | Vendor WMNS or 3rd-party overlay systems — evaluate and supply pertinent data |
| INVESTIGATE | Wi-Fi Scanners, Built-in Tools, CLI Commands, GUI Tools |
| MEASURE | Performance/throughput testing on Wireless and Wired — document where issues exist |
| VALIDATE | Survey software — map RF Coverage, visualize RSSI, SNR, CCI, Capacity; verify WLAN meets ALL requirements |
| ANALYZE | Layer 1 Spectrum Analysis — evaluate both Modulated and Unmodulated RF Interference |
| INTERROGATE | Frame Captures (Wired and/or Wireless) — understand communication flow details |

---

## 30-Row Potential Causes Table (Complete)
Three zones: WIRELESS (rows 1–15), LOCAL NETWORK (rows 16–28), INTERNET (rows 29–30)

### WIRELESS ZONE
| # | Type | Location | Potential Issues |
|---|------|----------|-----------------|
| 1 | Wireless | End User | Skills, Knowledge Perceptions, Device on/off, Understanding of Concepts & Device capabilities, Wi-Fi vs Cellular |
| 2 | Mobile | Wi-Fi Client Device | Drivers, Radio Capabilities, Profiles, Supported PHY, QoS, Power Save, Applications, Location, Vendor IE Support, Chipset Behavior, Roaming Algorithms, Auto-Negotiated MCS, MDM, Protection |
| 3 | | RF Media | RSSI, SNR, SNiR, Primary & Secondary Coverage, CCI/ACI, Retry Rates, Average MCS, Jitter, Latency, Consistency, Regulatory Domains, Non-Wi-Fi Interference, Spectrum Analysis |
| 4 | Per Frame Tx | Contention Process | Preamble Detect, Energy Detect, Triggers, NAV Timers, TxOP, AIFS, Random Slots, QoS, WMM, Duration ID, Ch Capacity, Non-Wi-Fi Interference |
| 5 | Per Frame Tx | MCS Process | Per Frame Decisions: Modulation, Coding, Ch Width, Guard Interval, Spatial Streams, Tx Power, ACK vs No ACK, TX decides |
| 6 | Per Time | Radio Resource Mgmt | Per Period Decisions: Channel, Tx Power, CCI, ACI, Noise, Duty Cycle, Retry Rates, CRCs, Load, Ch Width, DFS, User Traffic, Reg Domain, KPIs, Thresholds, Neighbor Discovery, Interference, Timing |
| 7 | Per Time | DFS Process | 802.11 NOT primary user. AP scans 60s, continuously scanning. If RADAR detected: send CSA, change channel. After 30-min can return (after 60-sec scan) |
| 8 | Per Frame Tx | Single Frame on RF | Overhead to deliver IP payload: AIFS, CW, BPSK Preamble, RTS, SIFS, Preamble BPSK, CTS, SIFS, Preamble, Preamble VHT, Header MBR, Payload PHY rate, CRC, SIFS, Preamble, ACK |
| 9 | Per Timers | Association Process | Beacon, Probe Req/Resp, Auth Req/Resp, Assoc Req/Resp. AP selection by: RSSI, SNR, Auth Method, Encryption, CSA, Error Ratios, MCS/Data Rates, Heuristics, Internal Lists, De-auth, Dis-assoc, 802.11 k,v,r, MBR, Proprietary |
| 10 | | 802.11 k,v,r | APs try to influence roaming decisions via 'standard' modes |
| 11 | Per Changes | Authentication Process | Open, PSK, 802.1X RADIUS. PSK = 4-Way Handshake → Encryption Keys. 802.1X = EAP Exchange → 4-Way Handshake |
| 12 | | Encryption Process | None, TKIP, AES/CCMP. Punishment for using TKIP. Confusion: WPA2 PSK is PSK-WPA2 |
| 13 | From LAN | Upper Layers | DHCP, IP, DNS, VLAN, Subnet Mask, Default Gateway, Captive Portal |
| 14 | | Controlled Port | AP controls which 802.11 frames can cross Wireless↔Wired boundary |
| 15 | Fixed | Access Point | Configurations, SSIDs, MBR, Supported PHY rates, Band Steering, Client Control, Radio Capabilities, Tx Rates, Client Isolation, Roaming, QoS. PoE, Antenna Pattern, Mounting, 1GB backhaul limit, AP Locations, Physical Layer, Firmware, RRM/ARM, Proprietary |

### LOCAL NETWORK ZONE
| # | Location | Potential Issues |
|---|----------|-----------------|
| 16 | Wired Medium | EIA/TIA 568A/B, Category Mismatch, Validation Tests, Grounding |
| 17 | Edge Switch | VLANs, Port Speeds, PoE, Configurations, QoS, End-to-End?, COS vs DSCP |
| 18 | Local Network | Distributed vs Centralized Forwarding, ACLs, VLANs, QoS, Tunnels, Layers, NAT |
| 19 | TCP/UDP | All TCP issues + UDP reasons for using each |
| 20 | Quality of Service | Tagged vs Untagged Port, DSCP, WMM Categories, End-to-End QoS |
| 21 | Applications | MTU, TCP Window, Round Trip Time, Processing Time, TCP Retransmission times |
| 22 | DHCP Server | Lease Durations, Broadcast Storms, Latency, Performance, Address Pool Scopes, Scalability, DHCP Options, Auto Renew |
| 23 | DNS | Configuration, Scalability, Security, Accuracy, Customization, Control, Blacklists |
| 24 | 802.1X/RADIUS | Configuration, Ports, Ranges, Licensing, EAP types, Custom VSA, Scalability, Resources, Certificate Issues, Fast/Secure Roaming types |
| 25 | Active Directory | Accounts, Credentials, EAP Compatibility, Custom RADIUS Attributes |
| 26 | Controller Functions | Code Versions, Bugs, Configurations, Local vs Cloud, Licensing, Distributed vs Centralized Forwarding, VLAN choices |
| 27 | Firewall | Firewall Rules, Capacity, Compatibility, Rate Limiting, Bandwidth Shaping |
| 28 | WAN Router | Size of Internet Pipe, Internet Destination Issues, Costs, Availability, Consistency |

### INTERNET ZONE
| # | Location | Potential Issues |
|---|----------|-----------------|
| 29 | Internet Connection | Bandwidth Throttling, Jitter, Latency |
| 30 | Captive Portal | Security, Client Issues, Privacy, Friction, Triggers, Certificates, DNS, Location, Control, Monetization, Legal, MiFi |

---

## Frame Distribution Stats (from slide 13)
- 802.11 Control frames: **60%**
- 802.11 Management frames: **15%**
- 802.11 Data frames: **25%**

## Frame Size Distribution (from slide 14)
- Less than 256 bytes: **80%**
- More than 512 bytes: **15%**
- More than 1024 bytes: **5%**

**Implication**: Most WiFi traffic is small frames (management + control overhead). This is why OFDMA (scheduling multiple small frames simultaneously) is so impactful — most frames are already small enough to fit in Resource Units.

---

## Key Deployment Contexts Shown in WLPC 2026
- Wired Before/After: Physical cabling cleanup is prerequisite to good WiFi
- Wireless Before/After: Room with no visible APs (ceiling-mounted, hidden) — WiFi 7 era AP placement
- Extreme deployments: Nuclear power plants, Kawangware (Kenya), Wi-Fi in space (Blue Origin rocket)
- WLANPros humanitarian work: School deployments in Kenya (Kawangware) — WiFi for education
