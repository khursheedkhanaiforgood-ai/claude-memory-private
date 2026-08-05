---
name: WiFi Digital Twin — Design Parameter Manifest (DPM)
description: Complete parameter set for the Intake Agent DPM. 7 sections (A-G), ~60 parameters. Socratic dialogue → DPM review → user calibration → design engine runs. Three source types: User provided / Derived / Default-confirm.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Pattern
Socratic dialogue → Design Parameter Manifest (DPM) → user calibrates → Design Engine runs.
DPM is the handoff artifact: Intake Agent → Design Engine.

## Three Source Flags
- [User provided] — from dialogue
- [Derived from X] — calculated from another input
- [Default — confirm?] — system assumption, needs calibration

## Section A — Site & Environment
| Parameter | Source | Why |
|-----------|--------|-----|
| Site type | User | RF propagation model |
| Coverage area m² | User | Min AP count for coverage |
| Number of floors | User | Multiplies area |
| Ceiling height (m) | User / default 3m | Effective coverage radius |
| Wall type | User (open/light/heavy/concrete) | Attenuation → path loss |
| RF environment type | User / derived | Noise floor + available airtime baseline |
| Noise floor (dBm, 20 MHz) | Derived / override | SNR = RSSI − noise floor |
| Available airtime % | Derived / override | Max usable channel capacity |
| Regulatory domain (country) | User | Legal channels, max EIRP, DFS |
| U-NII bands available | Derived from reg. domain | Non-overlapping 5 GHz channel count |

## Section B — Client Devices (per device type row)
| Parameter | Source | Why |
|-----------|--------|-----|
| Client device type | User | Protocol, SS, antenna gain lookup |
| 802.11 protocol | User / library | MCS ceiling |
| Tx antenna chains (= SS) | User / library | Peak data rate ceiling |
| Rx antenna chains | User / library | MRC uplink diversity gain |
| Antenna gain 2.4 GHz (dBi) | User / library default | SNR → MCS |
| Antenna gain 5 GHz (dBi) | User / library default | Same, 5 GHz |
| Max channel width | User / library | Legacy device channel width limit |
| Frequency band support | User | Client distribution calculation |
| Device quantity | User | Total in area (all: assoc + unassoc) |
| Concurrent association % | User / NMS / default 75% | % concurrently on WLAN |
| Concurrent active % | User / NMS / default 50% | % actively passing traffic |
| Concurrent limit mode | User (Both/Assoc/Active/None) | How % applies to this device type |

## Section C — Applications (per device type)
| Parameter | Source | Why |
|-----------|--------|-----|
| Application type | User / library | Throughput + frame characteristics |
| Throughput SLA (Kbps/Mbps) | User / library | Numerator of airtime formula |
| Transport (TCP/UDP) | Derived / override | TCP ACK overhead |
| Latency sensitivity | Derived from app | High latency = no aggregation = more overhead |
| Average frame size (bytes) | Derived / override | VoIP small = high overhead |
| Background app flag | User | Multi-app same device — adds airtime not device count |

## Section D — Network Design
| Parameter | Source | Why |
|-----------|--------|-----|
| AP model | User | Spatial streams, bands, PoE, SDR |
| 5 GHz channel width | User | Throughput vs channel reuse trade-off |
| 6 GHz channel width | User | PSC channel count |
| Client band distribution % | User / auto | 2.4/5/6 radio count ratio |
| Association limit per radio | User / default 128 | Association vs airtime bottleneck |
| Number of enabled SSIDs | User | Beacon overhead = SSID × AP × rate |
| MBR 2.4 GHz (Mbps) | User / default 12 | Cell size + legacy exclusion |
| MBR 5 GHz (Mbps) | User / default 12 | Same |
| MBR 6 GHz (Mbps) | User / default 12 | Same |
| RF coverage tier | User (Capacity/Data/Basic) | −67/−75/−80 dBm guarantee |
| Capacity for growth % | User / default 15% | Reserve headroom |
| 2.4 GHz radio limit | User / derived | Max 3 non-overlapping per area |
| Growth band preference | User / default 5 GHz | Where growth capacity lives |

## Section E — Channel Planning
| Parameter | Source | Why |
|-----------|--------|-----|
| 2.4 GHz channels | Fixed 1/6/11 | ACI worse than CCI |
| 5 GHz channel set | Derived from reg. domain | Channel count → reuse distance |
| DFS usage | User | Doubles channels; radar events disrupt |
| 6 GHz PSC channels | System default | Out-of-band discovery |
| CCI threshold (dBm) | User / default −85 | DCS trigger + capacity |
| ACI threshold (dBm) | User / default −85 | Worse than CCI |
| Consistent channel bonding | Fixed: high side | Reduces client confusion |

## Section F — Security & SSID
| Parameter | Source | Why |
|-----------|--------|-----|
| Security type per SSID | User | WPA3 mandatory 6 GHz |
| 802.11r FT | User / default ON | Fast roaming |
| 802.11k Neighbor Report | User / default ON | Roaming intelligence |
| 802.11v BSS Transition | User / default ON | AP-guided roaming |
| 802.11w MFP | Required 6 GHz | Management frame protection |
| VLAN per SSID | User | Switch port/trunk config |
| Client isolation | User | Guest/public networks |

## Section G — RRM & AutoRF
| Parameter | Source | Why |
|-----------|--------|-----|
| RRM mode | User (XIQ/static/per-AP) | Which RRM layer active |
| XIQ cloud RRM | User / default enabled | Holistic global — preferred |
| Broadcom per-AP AutoRF | User / default fallback | Local fallback on cloud outage |
| AutoRF CCI trigger (dBm) | User / default −85 | DCS channel change trigger |
| AutoRF retry rate trigger % | User / default 20% | DCS trigger |
| AutoRF channel util trigger % | User / default 70% | DCS trigger |
| TX power min (dBm) | User / default 7/13/16 | Prevents coverage holes from TPC |
| TX power max (dBm) | User | Prevents excessive CCI |
| Dynamic channel width | Fixed: DISABLED | WLANPros ❌ — breaks roaming |

## DPM UX Rules
1. All [Default—confirm?] fields shown with ⚠ warning
2. Derived values show derivation chain (e.g. "−92 dBm derived from Open Office RF environment")
3. User can edit ANY field before confirming
4. After edit: affected derived values auto-recalculate
5. SSID beacon overhead auto-shown after SSID count entered
6. Concurrent active % must be ≤ concurrent association % (validated)
7. Teaching Agent explains any field the user questions
