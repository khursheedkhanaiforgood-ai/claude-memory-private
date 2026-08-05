---
name: Revolution Wi-Fi Capacity Planner — Complete Methodology
description: Full content of Revolution Wi-Fi Capacity Planner User Guide (pp.1-38) and SSID Overhead Calculator — formulas, input fields, output structure, design rules. Private, not for GitHub.
type: reference
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Source Files
- `Support Documents/Revolution_Wi-Fi_Capacity_Planner_v2/Revolution Wi-Fi Capacity Planner User Guide.pdf`
- `Support Documents/Revolution_Wi-Fi_Capacity_Planner_v2/Revolution Wi-Fi Capacity Planner v2.0 (with 30 client rows).xlsm`
- `Support Documents/Revolution_Wi-Fi_Capacity_Planner_v2/Wi-Fi SSID Overhead Calculator.xlsx`
**PRIVATE — never commit to GitHub**

---

## Fundamental Design Principles

- True WLAN capacity = **airtime demand** (not device count, not coverage area)
- Two capacity dimensions: **Association Capacity** + **Airtime Demand**
- **MANDATORY**: Capacity Planner MUST be used with an RF coverage planning tool — NEVER alone
- RF coverage tools cannot forecast capacity (coverage-based only, miss airtime model)
- Even isolated environments: clients only achieve **80-90% airtime** due to contention overhead
- Design spectrum: Basic Connectivity ↔ Balanced Design ↔ Maximum Capacity

## Core Airtime Formula

```
Airtime Utilization = Application Throughput / Device Throughput Capability
```
- Device Throughput Capability varies by application (not just bandwidth)
- VoIP example: Cisco 792x G.711, 128 Kbps → assumed 0.64% but actual **1.68%** (2.5× higher)
- Because VoIP = small frames + high framing overhead + no aggregation
- Impact: single radio supports ~27 voice calls (NOT ~72) at <50% channel utilization

## Available Airtime Formula

```
Available Airtime = RF Environment Airtime − SSID Beacon Overhead
```

## AP Radio Count Formula

```
AP Radios Required = Σ(Active Devices × Airtime per Device) / Available Airtime per Radio
```

---

## Worksheet Structure (4 Sections)

### Section 1: Network Design Inputs

| Input | Description | Key Values |
|-------|-------------|-----------|
| AP Type | AP hardware capabilities (protocol, spatial streams, antenna gain) | Determines achievable data rates |
| 5 GHz Channel Width | Channel width for capacity calculations | 20/40/80/160 MHz |
| Client Distribution | % of dual-band clients on 5 GHz vs 2.4 GHz | Auto-adjusted by Capacity Distribution method |
| Association Limit per AP Radio | Max clients per radio | AP hardware cap: 128–256; artificially lower → may require more APs for association than airtime |
| Concurrent Associated Client % | % of all devices concurrently associated | Use NMS historical data; example: 150 seats, 100 connected = 66.67% |
| Concurrent Active Client % | % of devices actively passing traffic | Must be ≤ Associated %; idle devices = VoIP carts, printers, tablet carts |
| Number of Enabled SSIDs | SSIDs feeding beacon overhead calc | Directly feeds SSID Overhead worksheet → reduces available airtime |
| Min Basic Data Rate (2.4 GHz) | MBR for beacons + broadcasts (2.4) | Must be 802.11a/b/g rate even on 802.11n/ac; note: assumes all lower rates disabled |
| Min Basic Data Rate (5 GHz) | MBR for beacons + broadcasts (5) | Same constraint |
| RF Coverage Design | Minimum guaranteed RSSI in coverage | Capacity/Voice/Location = **-67 dBm**; Data Coverage = **-75 dBm**; Basic Connectivity = **-80 dBm** |
| RF Environment | Noise floor + available airtime type | Noise floor → SNR → data rates → airtime; available = total − ambient channel utilization |
| Capacity for Growth | % total capacity reserved for growth | Rounds up to next whole AP radio; "minimum guaranteed capacity for growth" |
| Device Sub-Total to Display | Which metric to show per device row | Airtime/Radio; AP Radios Reqd (Airtime); AP Radios Reqd (Association); Max Devices per Radio |

**RF Environment notes:**
- Noise floor measured for **20 MHz channel**; every doubling of width adds 3 dB (Capacity Planner accounts for this)
- Lowest valid noise floor: **-101 dBm**
- Measure via spectrum analysis: -85 dBm threshold for WiFi signals, -65 dBm for non-WiFi
- For custom environments: measure channel utilization on idle AP or monitor-mode AP to isolate ambient

**2.4 GHz growth:** In most environments, 2.4 GHz cannot add capacity (overlapping channels share, not add). Pin all growth to 5 GHz band.

---

### Section 2: Capacity Demand (per-device rows)

Each row = one Client Device × one Application (or throughput SLA)

| Field | Description |
|-------|-------------|
| Client Device | From Clients worksheet: 802.11 protocol, channel width, band restrictions, spatial streams, antenna gain, MRC |
| Application or Throughput SLA | From Applications worksheet: determines throughput target; VoIP, web browsing, video, etc. |
| Device Quantity | Total devices in physical area (ALL: associated + unassociated, active + idle) |
| Concurrent Limits | Which concurrency limits apply (see table below) |
| Background App Checkbox | Same device running additional app — airtime added, device NOT double-counted |

**Concurrent Limits options:**

| Option | Use Case | Behavior |
|--------|----------|----------|
| Both (Blank) | Laptops, tablets — mobile, interactive | Both association and active limits apply |
| Association | Video carts, powered-on-means-in-use devices | Association limited; active = association qty (always active when connected) |
| Active | Laptop carts, VoIP phones, inventory scanners | Active limited; full qty associates (idle but connected) |
| None | Dedicated IP cameras | All devices always associated AND always active |

**Informational fields per row (auto-calculated):**
- Application Throughput (reference)
- Association Quantity per band
- Active Quantity per band
- Airtime per device per band

---

### Section 3: Capacity Calculation Options

**Capacity Distribution methods:**
1. **Manual** (default): use Client Distribution % from Network Design
2. **Evenly** between 2.4/5 GHz: only when all AP radios enabled, be careful of 2.4 GHz CCI
3. **By available channels** (recommended): distributes across all non-overlapping channels → maximizes channel reuse, reduces CCI; heavier 5 GHz deployment

**2.4 GHz radio limit:** Caps total 2.4 GHz radios (useful when physical coverage allows only N non-overlapping 2.4 GHz channels)

**AP Form-Factor options:**
1. Dual-Radio (all enabled): equal 2.4+5 GHz radio counts
2. Dual-Radio (some disabled): disable 2.4 GHz radios where CCI would degrade capacity
3. Mix of Dual + Single-Radio: coverage via dual-radio + capacity boost via single-radio 5 GHz APs
4. Other (radio quantities only): for tri-radio APs (WiFi 6E/7 with 2.4+5+6 GHz)

---

### Section 4: Capacity Results

**Top-level outputs:**
- Total / Associated / Active devices (per band)
- AP Radios Required (airtime demand) per band
- AP Radios Required (association capacity) per band
- Capacity for Growth (additional radios, per band)
- Total AP Radios to Deploy (demand + growth, per band)
- Total Throughput required (for WAN/backhaul sizing)
- Band Steering ratio (% clients steered to 5 GHz)
- AP deployment recommendation (based on form-factor)

**Example output (Figure 12 — 700 devices):**
```
Total Devices: 700
Associated: 462 (2.4GHz: 66, 5GHz: 396)
Active:     352 (2.4GHz: 48, 5GHz: 304)
AP Radios Required: 2.4GHz = 2.95, 5GHz = 7.84
+ Growth (15%):     2.4GHz = 0.00, 5GHz = 1.90
AP Radios to Deploy: 2.4GHz = 3, 5GHz = 10
Recommendation: 10 Dual-Radio APs, 7 2.4GHz radios disabled
Total Throughput: 1.058 Gbps
Band Steering: 64.3%
```

**Capacity Breakdown per radio:**
- Available Channels (per band at selected channel width)
- Available Airtime (per channel after SSID overhead subtracted)
- Associated/Active devices per radio average
- Throughput per radio average

---

## SSID Overhead Calculator

**File:** `Wi-Fi SSID Overhead Calculator.xlsx`
**Sheet:** "Beacon Airtime Utilization"

**Default beacon parameters:**
- Beacon Data Rate: 802.11g 6 Mbps
- Frame Size: 380 bytes
- Beacon Interval: 102.4 ms

**Overhead bands:**
| Range | Classification |
|-------|---------------|
| 0–10% | Low |
| 10–20% | Medium |
| 20–50% | High |
| >50% | Very High |

**Key lookup values (APs on channel × SSIDs):**
| APs | 1 SSID | 4 SSIDs | 10 SSIDs |
|-----|--------|---------|---------|
| 1   | 0.56%  | 2.2%    | 5.6%    |
| 5   | 2.8%   | 11.1%   | 27.8%   |
| 10  | 5.6%   | 22.2%   | 55.6%   |
| 18  | 10%    | 40%     | 100%    |

**Design rule:** ≤4 SSIDs per band is NOT just a preference — at ≥10 APs on channel, 4 SSIDs = 22% overhead. Beyond 6 SSIDs, overhead becomes High or Very High with any real AP density.

---

---

## Link Budget Engine (Clients Worksheet + Data Rates Sheet)

This is the WiFi equivalent of the downlink/uplink link budget. It lives across two sheets.

### Data Rates Sheet — SNR→MCS Lookup Table
Two-dimensional table: SNR (0–50 dB) × Protocol+Channel Width → achievable MCS index
Protocols covered: 802.11b, 802.11a/g, 802.11n (1-4SS × 20/40 MHz × 800/400ns GI), 802.11ac (1-4SS × 20/40/80/160 MHz × 800/400ns GI)
Also contains: full data rate tables per MCS index (Modulation, Coding Ratio, Bits/Symbol, Mbps per channel width and GI)
Reference sensitivity: Minimum = -88 dBm @ SNR=0; Common = -93 dBm @ SNR=0

### Clients Worksheet — Connection Details (per device type)

**Client Device Capabilities columns:**
- 802.11 Protocol (b/a/g/n/ac)
- Tx Antenna Chains (= Spatial Streams)
- Rx Antenna Chains (Rx > Tx = MRC uplink diversity gain)
- Antenna Gain dBi (2.4 GHz and 5 GHz separately)
- Max Channel Width
- Frequency Band Support (Both / 2.4 GHz Only / 5 GHz Only)
- Maximum Data Rates at 20/40/80/160 MHz (from Data Rates sheet VLOOKUP)

**Connection Details columns (link budget outputs, per band):**
| Output | How Computed |
|--------|-------------|
| 802.11 Protocol | Constrained by WLAN config (Network sheet) |
| Spatial Streams | min(client Tx chains, AP SS limit from Network sheet) |
| Channel Width | 20 MHz for 2.4 GHz; configurable for 5 GHz |
| **SNR** | RF Coverage Design RSSI − RF Environment noise floor (± channel width noise correction) |
| **Link Data Rate** | Data Rates sheet: SNR → MCS index → data rate (Mbps), constrained by SS + channel width |

### Complete Link Budget Chain

```
RF Coverage Design RSSI tier (−67/−75/−80 dBm)
    − RF Environment noise floor (e.g. −92 dBm typical office, 20 MHz)
    = SNR estimate (e.g. 25 dB)
    → Data Rates sheet: SNR × (Protocol + Channel Width) → MCS index
    → Data rate at MCS × min(client SS, AP SS limit) × channel width
    = Device Throughput Capability (= Link Data Rate)
    → Airtime per device = App Throughput / Device Throughput Capability
```

### Uplink vs Downlink
- **Downlink**: AP TX power → propagation → client SNR → MCS → DL data rate (above chain)
- **Uplink**: Client antenna gain + Rx chains at AP → MRC diversity gain
  - Rx chain count > Tx chain count in Clients worksheet = UL receive gain (MRC)
  - Higher Rx chains at AP = better UL sensitivity

### Architectural Difference from 5G Link Budget

| 5G | WiFi (Capacity Planner) |
|----|------------------------|
| EIRP → path loss model → SINR → MCS → throughput | Required RSSI (design target) → noise floor → SNR → MCS → throughput |
| Link budget determines cell coverage radius | Coverage radius determined by RF survey tool (Ekahau/Hamina) — SEPARATE from this tool |
| Cell count from link budget | AP count = max(coverage-driven, capacity-driven) |

**Critical rule: AP count = max(coverage-driven AP count, capacity-driven AP count)**
- Capacity Planner solves the **capacity** side only
- RF survey tool solves the **coverage** side
- You iterate between them until both constraints satisfied (see Iterative Loop section below)

---

## Integration into Digital Twin

These inputs/outputs map directly to the Capacity Planning Sub-agent:

**Inputs from Intake Agent:**
- Client device types + quantities + capabilities
- Applications + throughput SLAs
- AP type selection
- Site area + environment type
- SSID count plan

**Capacity Planning Sub-agent outputs:**
- AP Radio count required (airtime) per band
- AP Radio count required (association) per band
- Binding constraint (airtime vs association — take the max)
- Available airtime per radio (feeds Simulation Agent)
- Total backhaul throughput required
- SSID beacon overhead (% of channel)
- Iterative loop: if 5 GHz radios required > what RF coverage plan places → revise channel width or cell sizing → re-run

**Key WiFi 7 / 6 GHz extension:**
The Capacity Planner v2 only covers 2.4+5 GHz (pre-WiFi 6E era). For 6 GHz:
- Add third band column with same methodology
- Available channels: 6 GHz has 59 non-overlapping 20 MHz channels (US regulatory)
- Available airtime: 6 GHz starts clean (no legacy devices, no CCI from incumbents initially)
- Form-factor: "Other (radio quantities only)" for tri-radio APs (AP4000, AP5020, AP5022, AP5060)
