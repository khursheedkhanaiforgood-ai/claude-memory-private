---
name: Digital Twin Live AP Calibration Architecture — June 5 2026
description: Full verbatim architecture dialogue for DigitalTwinEngine: APObserverAgent + SimulatorAgent + FederatedTwinAgent. Branch feature/live-ap-monitor. Calibration loop design, 15-param bridge, Monte Carlo sensitivity oracle, QoE proxy vector. For EOD HTML.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---

## Context
Session: June 5 2026. Continuation of AP CLI diagnostics + live monitor planning.
Branch: `feature/live-ap-monitor` (created this session from main, commit aebe9e4)
Repo: `/Users/khukhan/5320-onboarding-agent`

---

## VERBATIM DIALOGUE (for EOD HTML link-list)

---

**User:** So, can we create a new branch to do this pls? I don't want to touch what we already have. Also, can we not use an agentic approach here with a lead agent_orchestrator running the sub-agents and another agent from the simulator we are creating?

**Claude:** Good. The schema already handles this — `radio` is a string column. The key design decisions: [multi-radio architecture response with wifi0/wifi1/wifi2 auto-detection, dynamic radio tabs, Mode A/B display options]

---

**User:** I want for both 2.4GHz and 5GHz now.. with option to add 6GHz

**Claude:** Good. The schema already handles this — `radio` is a string column [...] The collector tries all three every poll. If wifi2 doesn't exist, it silently skips. When you upgrade to a WiFi 7 AP, it auto-appears with zero code change. [schema with band column, dynamic radio tabs, Mode A side-by-side / Mode B overlay]

---

**User:** Yes, option A sprint_2, option B for sprint_3. Now, I want to make sure I understand this clearly. We have new branch with AP. This branch will be displaying, storing and in the future communicating with the simulator. First, I want to make sure, we can grab, plot all the relevant time-series and other data from AP. Then, plot based on user needs, but store everything in local drive for regression. Then, I want to make sure given the inputs from the AP-plots to the simulator, 'it provides its QoE'... now how will it do that? I give 60-parameter inputs to the simulator with 15 of them affecting change. What will the AP-plots (real data) give to simulator? The AP is already configured with those parameters. So, we will need some kind of a 'federated agent' which will sit on top of the simulator_AP pair (what are we calling this AP plot agent) which will help 'calibrate' the Simulator_15parameters then feed into the AP then see what the QoE is and formulate delta to fix? Now the simulator already has monte-carlo running against the nominal points so? Do you see how I am thinking? PLS STORE ALL THIS DIALOGUE IN MEMORY WITH YOUR RESPONSES VERBATIM SO you can link-list in EOD HTML.

**Claude:**
Yes, I see exactly how you're thinking — and it's a clean closed-loop Digital Twin architecture.

### The Calibration Loop

```
FederatedTwinAgent (sits above the AP ↔ Simulator pair)

1. Real AP data → map to 15 simulator params
2. Run simulator with calibrated params → predicted QoE
3. Measure real QoE from AP outputs
4. Compute delta (predicted − real)
5. Use Monte Carlo sensitivity (∂QoE/∂param_i) to rank which of the 15 params to adjust first
6. Adjust → re-run → iterate until delta closes
```

### The 15-Parameter Bridge

| Real AP Measurement | → Simulator Parameter |
|---|---|
| Station count | `n_clients` |
| Channel width (wifi1) | `chan_width_mhz` |
| CRC error % | `collision_rate` |
| Noise floor (dBm) | `noise_floor_dbm` |
| Avg SNR per client | `snr_mean_db` |
| Tx airtime CU% | `tx_utilization` |
| Rx airtime CU% | `rx_utilization` |
| Interference CU% | `interference_cu` |
| Avg Tx rate (MCS proxy) | `mcs_tier` |
| Negotiated chan-width per client | `actual_chan_width_client` |
| ACSP neighbor count | `co_channel_neighbors` |
| Strongest neighbor RSSI | `external_interference_rssi` |
| Summary state (collision/fair/good) | `collision_state` |
| Peak Tx Mbps | `peak_throughput_measured` |
| Station VLAN distribution | `traffic_mix` (app profile proxy) |

The other 45 simulator parameters (site geometry, AP placement, building materials) are **static config inputs** from the DPM. The 15 above are the ones real AP data can populate dynamically.

### The Monte Carlo Connection

The Monte Carlo already computes variance around nominal points — that variance IS the sensitivity oracle: `∂QoE/∂param_i` for each of the 15 parameters.

When delta (predicted − real) is non-zero, the FederatedTwinAgent doesn't guess blindly. It reads the MC sensitivity table and adjusts the highest-leverage parameter first. This is **gradient-informed inverse calibration** — not brute force.

### QoE From Real AP Data (5-dimensional proxy vector)

| QoE Dimension | Real AP Proxy |
|---|---|
| Throughput | Avg Tx rate Mbps per client |
| Reliability | 1 − CRC% |
| Latency (proxy) | Airtime utilization → M/D/1 model |
| Connection quality | SNR → MCS tier |
| Capacity headroom | 1 − (Tx CU% + Rx CU%) |

### Agent Names Final

| Agent | Role |
|---|---|
| `APObserverAgent` | Collection + parsing + plotting |
| `SimulatorAgent` | Python port of the 15-equation QoE engine |
| `FederatedTwinAgent` | Calibration loop — owns the delta |
| `OrchestratorAgent` | Coordinates all agents, drives 5s cycle |
| `StorageAgent` | PostgreSQL writes (ap_metrics, client_metrics, sim_vs_real) |
| `DashboardAgent` | Dash app on Railway — serves colleagues |

Full system name: **DigitalTwinEngine** (Rung 2 of the Digital Twin architecture).

---

**User:** Yes, for now, I will run the collector manually on APs in my lab. For the future, we can create a widget that runs on my MacBook and link to Railway which can 'start the widget'?

**Claude:**
Exactly right — and the pattern for that already exists.

### Now (Sprint 2) — Manual
```
You → terminal → python collector.py → Railway PostgreSQL ← Railway Dashboard (colleagues)
```

### Future (Sprint 3) — Remote-Start Widget
```
Railway Dashboard
  [▶ Start Collector] button
         ↓ writes to PostgreSQL
  collector_control table {status: 'start', ap: '192.168.0.12'}
         ↑ polls every 10s
  macOS Menu Bar Widget (rumps)
  🟢 Collecting · AH-556680 · 5GHz · CRC 13%
         ↓ spawns subprocess
  collector.py → SSH → AP
```

PostgreSQL is the control plane — no open ports needed on Mac, no ngrok, no tunnels. Widget polls the same DB the dashboard already has.

### `collector_control` table (add now, use in Sprint 3)
```sql
CREATE TABLE collector_control (
  ap_ip      VARCHAR(15) PRIMARY KEY,
  ap_name    VARCHAR(50),
  status     VARCHAR(10) DEFAULT 'idle',  -- 'idle', 'start', 'stop'
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## Architecture Decisions Locked

- Branch: `feature/live-ap-monitor`
- Sprint 2: Option A (synchronous agents, manual collector start)
- Sprint 3: Option B (async queue + macOS menu bar widget + Railway remote-start)
- Store EVERYTHING every poll — selector controls VIEW not STORAGE
- Radios: wifi0 (2.4GHz) + wifi1 (5GHz) + wifi2 (6GHz, auto-detect, skip if absent)
- Railway PostgreSQL: 4 tables (ap_metrics, client_metrics, sim_vs_real, collector_control)
- Display: Mode A (side-by-side panels, default) → Mode B (overlay, Sprint 3)

## Directory Structure
```
live_monitor/
├── agents/
│   ├── orchestrator.py    ← OrchestratorAgent
│   ├── collector.py       ← APObserverAgent / CollectorAgent
│   ├── storage.py         ← StorageAgent
│   ├── dashboard.py       ← DashboardAgent
│   └── simulator.py       ← SimulatorAgent (Python QoE equations)
├── schema.sql
├── config.py
├── main.py
└── requirements.txt
```

## Why: Key Decisions
- FederatedTwinAgent uses MC sensitivity as gradient oracle → closes delta fastest
- 15 params = observable AP outputs; 45 params = static DPM inputs; not all 60 come from AP
- QoE is a 5-vector (throughput/reliability/latency/connection quality/headroom) not a scalar
- `collector_control` table added now (Sprint 2) so Sprint 3 widget has schema to plug into

---

## VERBATIM DIALOGUE CONTINUED

**User:** Hold. You know the 60parameters. You know the 15. What are the 15 parameters that map to the actual AP_parameters, share a table. Now, do the same for the rest of the 45. Where they don't share anything, leave blank on the AP side. Also, for the delta between predicted-observed, can we keep the QoE dimensions open, and choices given to the user with the 5-dimensional standard one as the default? BTW, make sure this work from today on the twin_calibration is completely independent so we don't disrupt our v3 of the simulator work?

**Claude:**
[Built two authoritative tables from DPM memory + track_b calibration memory]

### 15 Search Knobs → AP CLI (the parameters the simulator optimizes AND the AP is configured with)

| # | DPM | Parameter | AP CLI Command | Current AH-556680 |
|---|---|---|---|---|
| 1 | D2 | 5 GHz channel width | `show interface wifi1 → Channel width=` | 80 MHz ✓ |
| 2 | D3 | 6 GHz channel width | `show interface wifi2 → Channel width=` | N/A |
| 3 | D8 | MBR 2.4 GHz | `show radio profile radio_ng_11ax-2g → Min basic rate=` | TBD |
| 4 | D9 | MBR 5 GHz | `show radio profile radio_ng_11ax-5g → Min basic rate=` | TBD |
| 5 | D10 | MBR 6 GHz | `show radio profile radio_ng_11ax-6g → Min basic rate=` | N/A |
| 6 | G8/9 | TX power min/max | `show interface wifi1 → Tx power=` | TBD |
| 7 | G1 | RRM / ACSP mode | `show interface wifi1 → ACSP=` | enabled |
| 8 | G5 | AutoRF CCI trigger | `show acsp channel-info → neighbor RSSI list` | -7 dBm (QFiberKK) |
| 9 | G6 | AutoRF retry rate trigger % | `show interface wifi1 → CRC error %=` | 13% |
| 10 | G7 | AutoRF channel util trigger % | `show interface wifi1 → Tx CU% + Rx CU%=` | TBD |
| 11 | G9 | Dynamic channel width | `show interface wifi1 → Dynamic channel width=` | enabled ✓ |
| 12 | G | OFDMA DL enabled | `show interface wifi1 → OFDMA DL=` | disabled |
| 13 | G | OFDMA UL enabled | `show interface wifi1 → OFDMA UL=` | disabled |
| 14 | G | MU-MIMO enabled | `show interface wifi1 → MU-MIMO=` | disabled |
| 15 | G | BSS Color enabled | `show interface wifi1 → BSS Color=` | disabled |
| +3 | — | Guard Interval | `show radio profile → GI=` | TBD |
| +3 | — | MCS Floor | `show radio profile → Min MCS=` | TBD |
| +3 | — | DTIM interval | `show radio profile → DTIM=` | TBD |

### 45 Static / Context DPM Parameters (blank where AP has no visibility)

Section A (Site): Noise floor + available airtime % have AP CLI proxies. Site type, area m², floors, ceiling height, wall type, regulatory domain = blank (user DPM only).

Section B (Clients): Device quantity, concurrent association %, concurrent active % observable from `show station`. Device type, antenna chains, protocol, antenna gain = blank.

Section C (Apps): All 6 app parameters (type, SLA, transport, latency sensitivity, frame size, background flag) = blank. AP has no app-layer visibility.

Section D (Network): AP model from `show version`. Client band distribution from `show station`. SSID count from `show ssid`. Association limit from `show radio profile`. Coverage tier, growth %, 2.4G limit = blank.

Section E (Channel): Current channel from `show interface wifi0/wifi1`. DFS channel usage from ACSP. CCI/ACI threshold values = blank (they're settings, not measurements).

Section F (Security): Security type + MFP + VLAN per SSID from `show ssid`. 802.11r/k/v flags from `show radio profile`. Client isolation from `show ssid`.

Section G (RRM): The 15 search knobs cover all relevant G params. TWT still disabled — not a search knob in Sprint 2.

### QoE Dimensions — User-Configurable Design

Default 5-vector (pre-selected):
1. Throughput → avg Tx Mbps per client
2. Reliability → 1 − CRC%
3. Latency (proxy) → M/D/1: qd = 6u/(1-u)
4. Connection quality → SNR → MCS tier
5. Capacity headroom → 1 − (Tx CU% + Rx CU%)

Optional user-addable (enabled when AP data supports it):
- Band utilization balance (observable now: 2.4 vs 5 vs 6 GHz split)
- Per-VLAN throughput (observable now: show station + VLAN filter)
- VoIP MOS (future: needs jitter + packet loss)
- POS success rate (future: needs rTWT, TWT currently disabled)
- Video buffer ratio (future: per-client traffic category data)
- Roaming latency (future: multi-AP data needed)

Delta vector = N-dimensional where N = active QoE dimensions. MC sensitivity pre-computed per dimension. FederatedTwinAgent adjusts the highest-leverage param for each dimension with non-zero delta.

### Independence Guarantee

`feature/live-ap-monitor` branch:
- `live_monitor/agents/simulator.py` = Python port of equations only
- ZERO imports from `docs/stadium_wifi_simulator_v3.html`
- v3 HTML lives on main, untouched
- Branch never merges back until Sprint 3+, only `live_monitor/` is new

---

---

## VERBATIM DIALOGUE CONTINUED — GS Convergence Protocol

**User:** Yes agreed but first, the observed-simulator delta should be configurable because how will the simulator 'converge' for delta to be zero? It would run forever? Now also, what is the mechanism by which the user can actually observe, outputs coming from AP, ingested by simulator, but first starting point with the golden_set simulator_gs_0 then go to AP_gs_0 ... see QoE_gs_0 then update QoE to where the user targets to see how the simulator changes simulator_gs_1, then we need to push this into AP_gs_1 and see QoE_AP_1. The QoE_AP_0 was there and we have QoE_Simulator_0 and 1 and so forth? Do you get the loop? Phase 4 perhaps? Final Sprint?

**Claude:**
Yes — I get the loop exactly. And you identified the critical design flaw: "delta = 0" is not a valid termination condition.

### Two Deltas — Never Conflate Them

| Delta | Measures | Who cares |
|---|---|---|
| δ_calibration_n = \|QoE_Sim_n − QoE_AP_n\| | Is the simulator model accurate? | FederatedTwinAgent / engineer |
| δ_progress_n = \|QoE_target − QoE_AP_n\| | Are we moving toward the goal? | User / operator |

Small δ_calibration + large δ_progress = model is correct but hardware change needed, not config.
Large δ_calibration = something in real AP not modeled (PPSK guest load, rogue neighbor) — PAUSE, don't iterate.

### GS Convergence Loop

```
GS_0 → Simulator_GS_0 → QoE_Sim_0 ─┐
GS_0 → Push to AP → QoE_AP_0 ───────┴─► δ_calibration_0

User sets QoE_target
FederatedTwinAgent proposes GS_1 (highest-leverage param change from MC sensitivity)
GS_1 → Simulator_GS_1 → QoE_Sim_1
[Human: push GS_1 to AP?] → QoE_AP_1
δ_progress_1 = |QoE_target − QoE_AP_1|
Continue? → GS_2 ...
```

### 5 Stopping Conditions (all user-configurable)

```python
CONVERGENCE_CONFIG = {
    'max_iterations':     5,     # never run more than N GS cycles
    'progress_threshold': 0.10,  # stop when δ_progress < 10% of target
    'min_improvement':    0.05,  # stop if GS_n→GS_n+1 gains < 5% (diminishing returns)
    'calibration_alarm':  0.20,  # PAUSE if δ_calibration > 20% — model unreliable
    'mode':               'HITL' # human approves every AP push; 'auto' = Sprint 5
}
```

Loop terminates on FIRST of: goal reached / max cycles / diminishing returns / manual accept / model drift alarm.

### Sprint Map
- Sprint 2 (NOW): APObserverAgent → collect, store, plot, Railway deploy
- Sprint 3: SimulatorAgent (Python equations) + FederatedTwinAgent + basic δ_calibration
- Sprint 4 (Phase 4 / Final): GS Convergence Protocol — full HITL loop, GS History Panel, convergence config UI
- Sprint 5: mode=auto (fully autonomous, XIQ API push — separate safety rails)

### GS History Panel (Phase 4 Dashboard)
Shows: GS_n timeline, QoE_AP vs QoE_Sim per iteration, δ_calibration, δ_progress, params changed per step.
Buttons: [Push GS_n to AP] [Accept GS_n] [Reset to GS_0] [Export]

### HITL Human Gate
Human always approves physical AP config push. Simulator proposes GS_n+1 automatically (highest-leverage param). Human decides whether to deploy. AP is production — never auto-deploy in Sprint 4.

---

---

## VERBATIM DIALOGUE CONTINUED — AP3000 Specifics

**User:** Hold. We have wifi0 and wifi1 in the AP now, with 3000. Make it specific. Pls save all of our dialogues in memory pls!

**Claude:**
Read actual session logs to extract confirmed field names. Built AP3000-specific tables.

### Key AP3000 Hardware Facts (confirmed from show interface wifi1)

- Model: AP3000, HiveOS, WiFi 6 (802.11ax), dual-band ONLY (wifi0=2.4GHz, wifi1=5GHz, no wifi2)
- Tx Chain=static 2; Rx Chain=static 2 → 2×2 MIMO, max 2 spatial streams BOTH radios
- One Chain EIRP power=22.00dBm(18dBm + 4.00dBi) → TX power = 18 dBm per chain
- Short guard interval=enabled → GI=400ns already ON
- Max clients number=100 → association limit per radio
- All WiFi 6 features DISABLED: HE OFDMA downlink=disabled, HE OFDMA uplink=disabled, MU-MIMO=disabled, BSS Color=0, TWT=disabled, Tx beamforming=disabled
- Dynamic channel width=enabled (after fix this session)
- Channel width=80MHz (wifi1, after fix)
- DFS=disabled; Frameburst=enabled (Aerohive-specific)
- Radio profiles: radio_ng_11ax-2g (wifi0), radio_ng_11ax-5g (wifi1)
- EDCA: AC_BE CWmin=4/CWmax=6, AC_VO CWmin=2/CWmax=3, AC_VI CWmin=3/CWmax=4

### AP3000 Hardware Constants for SimulatorAgent

```python
AP3000_HARDWARE = {
    'model':           'AP3000',
    'wifi_gen':        '802.11ax (WiFi 6)',
    'radios':          ['wifi0', 'wifi1'],  # no wifi2
    'spatial_streams': {'wifi0': 2, 'wifi1': 2},
    'max_chan_width':   {'wifi0': 40, 'wifi1': 80},  # MHz
    'max_mcs':          11,    # WiFi 6 only (not WiFi 7 MCS 13)
    'ofdma':            True,  # supported, currently disabled
    'mu_mimo_dl':       True,  # 2×2 DL only
    'twt':              True,  # supported, disabled
    'bss_color':        True,  # supported, BSS Color=0
    'max_clients':      100,   # per radio
    'tx_power_max_dbm': 18,    # per chain
    'bands_ghz':        [2.4, 5.0],  # no 6GHz
}
```

FederatedTwinAgent MUST respect these hardware ceilings when proposing GS_n+1:
- Never propose MCS 12/13 (WiFi 7 only)
- Never propose 160MHz channel width (AP3000 max = 80MHz on 5GHz, 40MHz on 2.4GHz)
- Never propose 4×4 MU-MIMO (max 2×2)
- Never propose wifi2/6GHz configs

### 15+3 Search Knobs — Exact HiveOS Field Names

Read command for all radio params: `show interface wifi0` / `show interface wifi1`
Read command for radio profile params: `show radio profile radio_ng_11ax-2g` / `show radio profile radio_ng_11ax-5g`

Confirmed field names from actual output:
- Channel width → `Channel width=80Mhz`
- TX power → `One Chain EIRP power=22.00dBm(18dBm + 4.00dBi)` (TX power = middle value)
- OFDMA DL → `HE OFDMA downlink=disabled`
- OFDMA UL → `HE OFDMA uplink=disabled`
- MU-MIMO → `MU-MIMO=disabled`
- BSS Color → `BSS Color=0` (0 = disabled)
- TWT → `TWT=disabled`
- Dynamic channel width → `Dynamic channel width=enabled`
- Guard Interval → `Short guard interval=enabled` (enabled = 400ns short GI)
- CRC proxy for retry % → `CRC error airtime percent=28.25%`
- CU% for AutoRF trigger → `Tx utilization=0%; Rx utilization=3%; Interference utilization=7%; Total utilization=10%`
- Noise floor → `Noise floor=-89dBm`

TBD (need next AP SSH session): MBR 2.4G/5G, MCS floor, DTIM interval, wifi0 OFDMA/MU-MIMO/BSS Color/TWT states

### AP3000-Specific Supplemental Params (collector captures always)

Frameburst=enabled (Aerohive-specific), A-MPDU=enabled, Beamforming=disabled,
EDCA params (CWmin/CWmax/AIFS per AC), BGSCAN count/missed, DFS state,
Beacon interval=100ms, Association limit=100

---

---

## VERBATIM DIALOGUE CONTINUED — AP Registration + Session DB + RAG

**User:** Should you not first ask about what AP we are working with from the user? Then somehow make sure that AP and configurable radios are 'in our database' and matches the 'simulator profile'? Or, we can have a drop down menu of APs to choose from and you will need to create a RAG database to store all docs as I gave you previously? Also, I want a database for all the data the user is going to poll to be stored by session so that we can go back to any regression testing and don't have to worry about figuring out what may have happened? The HITL analogy is good.

**Claude:**
Yes to all four. Phased approach:

### AP Onboarding Startup Flow
1. User provides AP IP + credentials
2. SSH → show version → auto-detect model + firmware + serial
3. Lookup ap_profiles table → load hardware constraints
4. If unknown model → prompt user to configure (or load datasheet)
5. Register/update ap_inventory
6. Snapshot all 15+3 search knob values into session record
7. Start polling loop

### Full Database Schema (6 tables)
- ap_profiles: hardware constants per model (AP3000, AP5060, AP4000, ...)
- ap_inventory: physical APs in lab (IP, name, model FK, firmware, location, last_seen)
- sessions: every collector run = UUID, ap_ip FK, gs_label, started_at, ended_at, notes, config_snapshot JSONB
- ap_metrics: session_id FK, ts, radio, band, all RF metrics
- client_metrics: session_id FK, ts, radio, mac, SNR, rates, etc.
- sim_vs_real: session_id FK, gs_label, qoe_measured JSONB, qoe_predicted JSONB, delta JSONB
- collector_control: ap_ip PK, status (idle/start/stop), updated_at

Session table is the KEY for regression testing — every data row links to a session, which has a config snapshot. Can replay any historical GS iteration.

### AP Dropdown in Dashboard
Reads ap_inventory. Session dropdown reads sessions WHERE ap_ip = selected. Load any historical session = full regression replay.

### RAG for AP Docs (Phased) — CORRECTED
Two separate RAG corpora:
1. EP1/XIQ corpus — already at /Users/khukhan/5320-onboarding-agent/docs/rag-corpus/ (26 docs: EP1 Networking/Security/AirDefense/Guest). NO AP specs.
2. AP Hardware corpus — NEW sub-dir: rag-corpus/ap-specs/ (EMPTY — needs AP spec sheets)
   - AP3000 datasheet + HiveOS CLI guide (immediate need — user to provide from Extreme internal)
   - AP5060 datasheet (future WiFi 7)
   - 802.11ax PHY rate table (MCS 0-11 × GI × chan width → Mbps)
NOTE: WLPC corpus (/Users/khukhan/Downloads/WLPC Troubleshooting/) is NOT for RAG — it is private troubleshooting files only.
- Sprint 2: static ap_profiles table row for AP3000 (seeded from confirmed CLI data, no RAG needed)
- Sprint 3: pgvector on Railway PostgreSQL. Index rag-corpus/ap-specs/ docs once user provides them.
- Sprint 4: FederatedTwinAgent queries AP hardware corpus for calibration reasoning.

### Revised Sprint Map
- Sprint 2 (NOW): CollectorAgent + ap_inventory + ap_profiles (AP3000) + sessions + StorageAgent + DashboardAgent (AP dropdown + session selector) + Railway deploy
- Sprint 3: SimulatorAgent + FederatedTwinAgent + pgvector RAG + macOS widget
- Sprint 4: GS Convergence Protocol — HITL loop + GS History Panel + CONVERGENCE_CONFIG UI
- Sprint 5: mode=auto + XIQ API push

---

## SPRINT 1 COMPLETE — June 5 2026

All Sprint 1 deliverables done and running:
- `live_monitor/agents/collector.py` — ~100 RadioMetrics fields, 5 commands per poll: `show interface wifi0/wifi1`, `show radio profile radio_ng_11ax-2g/5g`, `show station`, `show acsp channel-info`
- `live_monitor/dashboard.py` — Plotly Dash at localhost:8050+, 10 sidebar groups, time-series subplots per radio, boolean badge row, client DataTable. Fixed: em-dash bug in group IDs (use `_group_id()` with re.sub)
- `live_monitor/main.py` — developer entry point (rich.Live + Dash + browser + CSV)
- `live_monitor/widget.py` — macOS menu bar (rumps), menu bar icon shows 🟢/🟡/🔴 score
- CSV logs: `live_monitor/data/radio_{ts}.csv` + `client_{ts}.csv`
- `find_free_port()` auto-increments from 8050 if occupied

### show radio profile — confirmed field names (June 5 2026)
Actual output from AH-556680 radio profile:
- EDCA: `AC=be; WMM min CW=4; max CW=6; AIFS=3; txoplimit=0;` (BE, BG, VI, VO lines)
- `Weak SNR threshold=15 dB;` ← minRSSI/MCS floor proxy
- `LB station airtime limit=4%;` ← atFair proxy
- `Max ACSP tx power=20dBm;`, `Power floor=5dBm;`
- `Interference threshold=35%;`, `CRC error threshold=35%;`, `Channel utilization threshold=35%;`
- `High density=disabled;`, `Safety net=enabled;`, `Band steering=disabled;`, `Load balance=disabled;`
- `Dynamic channel width=enabled; Trigger threshold=medium;`
- ALL WiFi 6 features DISABLED: MU-MIMO=disabled, OFDMA dl/ul=disabled, TWT=disabled, BSS Color=0

### 12 Optimizer Knob Coverage (post radio profile parser)
From `OPT_SEARCH` in simulator JS (txPwr, bw5g, maxCli, cwBe, txopBe, mcsFloor, dtim, atFair, minRSSI, rxSop, fragThr, rtsThr):
- EXACT (6): txPwr→tx_power_dbm, bw5g→channel_width_mhz, maxCli→max_clients, cwBe→cw_min_be, txopBe→wmm_txop_be(=0), atFair→lb_airtime_limit_pct(=4%)
- PROXY (2): mcsFloor+minRSSI both → weak_snr_threshold_db(=15dB)
- PARTIAL (1): dtim → beacon_period_ms (beacon ≠ DTIM; DTIM not in AP3000 CLI)
- NOT EXPOSED (3): rxSop, fragThr, rtsThr — not in any AP3000 show command

### Sprint 2 Plan APPROVED (ready to implement)
Phase 1 (foundation): NEW storage.py (SQLiteStore, 4 tables: sessions/radio_polls/client_polls/knob_change_events), NEW report.py (generate_session_html), EDIT main.py (3 lines)
Phase 2 (knobs panel): NEW knob_panel.py (KNOB_SPEC + build_knob_panel), EDIT dashboard.py (daq.Gauge 4×3 grid), ADD dash-daq>=0.5.0
Phase 3 (Digital Twin bridge): NEW twin_bridge.py (write_twin_state, ChangeDetector, BaselineManager), Flask route /simulator + /twin-state.json, copied simulator in assets/
Architecture: SQLite+HTML report, dash-daq gauges, file-polling bridge, relative-delta-only QoE comparison, 12-poll stabilization window

## Session 2 Additions — June 5 Evening

### Three Bugs Fixed
- **IndexError in Dash chart callback**: Root cause = chart callback registered N checklist IDs as direct inputs. Browser/server count mismatch causes `flat_data[ind]` OOB. **Fix**: `dcc.Store(id='selected-metrics')` — `_update_store` aggregates all checklists, `_update_chart` has 2 inputs only. Permanent fix.
- **Frame Fail shows `?`**: Was checking `rm.tx_failed_pct` (always None on AP3000). Fixed to `rm.tx_error_pct`.
- **TWT `?` on wifi0**: Regex `^TWT=` missed leading whitespace. Fixed to `^\s*TWT\s*=\s*`.

### Definitions + Tooltips Added to dashboard.py
- `_DEFINITIONS` dict: 40 field → description + formula + AP CLI source
- `_FLAG_DEFINITIONS` dict: 14 feature flag → 802.11 explanation  
- `_CLIENT_COL_DEFS`: 14 client table column definitions
- `_metric_label()`: adds ⓘ with title to every sidebar metric
- `_badge(tooltip=)`: all status/feature badges have hover tooltip
- `tooltip_header` on DataTable: column header hover in client table
- Legend row always visible: `? = not reported · OFF = disabled · ON = enabled`

### Launch Sequence (canonical)
- **main.py**: Rich terminal table + browser dashboard + CSV. Run, then open localhost:8050.
- **widget.py**: macOS menu bar only, no terminal. Click icon → Open Dashboard.
- **DEBUG_CLI=true** in `.env` → raw AP SSH log at `live_monitor/data/debug_cli.log`. `tail -f` in second terminal.

## Session 3 Additions — June 5 Late Evening

### Tooltip Architecture (final)
- Problem: `title=` → native OS tooltip (invisible on Safari macOS). `dcc.Tooltip` → Dash 4.2.0 removed `target`/`placement` params entirely.
- **Fix**: `live_monitor/assets/dt_tips.js` — pure-JS floating div, `position:fixed` relative to viewport. Listens for `[data-tip]` attribute on any element. Escapes ALL overflow clipping. Auto-loaded by Dash from `assets/`.
- **Double tooltip bug**: Badges had both `className='dt-tip'` (CSS `::after`) AND `data-tip` (JS). Removed the CSS class from `_badge()`. Single tooltip source now.
- `live_monitor/assets/dt_tips.css` — styles the `#dt-floating-tip` div only (no `::after` rules).
- Sidebar ⓘ icon: `html.Span('ⓘ', **{'data-tip': defn})` in Checklist label. Works despite sidebar `overflow-y: auto`.

### OPEN QUESTION FOR TOMORROW — Degraded RF Injection
**How do we test degraded RF conditions or inject errors into the link to figure out how the 12 knobs should respond — without needing a real crowded venue?**

Options to evaluate tomorrow:
1. **iperf3 traffic load**: Run `iperf3 -c <client_ip> -t 300 -P 8` from a laptop → artificially loads Tx/Rx CU on the AP → AP reports higher `total_cu_pct`, lower throughput. Immediately available, no hardware.
2. **Co-channel interference AP**: Enable a second AP on the same channel (Ch 149 5GHz or Ch 11 2.4GHz) → AP ACSP sees it, `interference_cu_pct` rises. Already have infrastructure.
3. **Physical RF attenuation**: Wrap client in aluminum foil or move to edge of range → `snr_db` drops, `weak_snr_threshold_db` behavior observable. Zero cost.
4. **HiveOS `kdebug` injection**: HiveOS has kernel-level debug commands. Investigate whether any allow artificial CRC injection or error-rate forcing without PHY change. Needs AP3000 CLI research.
5. **Synthetic DataStore injection**: Add `--inject` mode to collector — reads a JSON "degraded scenario" file and pushes synthetic RadioMetrics to DataStore. Tests FederatedTwinAgent optimizer in software loop without any real AP changes. The cleanest for repeatable unit-testing of the optimizer.
6. **CSV replay**: Replay a saved session CSV at 5s intervals into DataStore — lets colleagues drive the dashboard without AP access.

**Recommendation (to discuss tomorrow)**: Implement synthetic injection (#5) first — it lets us test all 12 knob responses in software before touching the AP. Then layer in iperf3 (#1) for real RF stress tests. Co-channel AP (#2) for interference CU tests. kdebug (#4) last — needs investigation.

**Why this matters**: Without degraded RF, the optimizer always recommends "enable WiFi 6 features" (OFDMA, MU-MIMO) because the clean lab AP has headroom. In a real venue, the knobs are different. Synthetic injection bridges the gap between lab and stadium.

## Pending (build order — TOMORROW)
1. **Sprint 2 Phase 1** — START HERE: `storage.py` SQLiteStore → wire into `main.py` → verify DB populates + session HTML auto-generates
2. **Sprint 2 Phase 2**: `knob_panel.py` + dash-daq gauges → 4×3 grid in dashboard
3. **Sprint 2 Phase 3**: `twin_bridge.py` + /simulator route + live AP sidebar in v3 simulator copy
4. **Railway deploy** — architecture decision first: local collector → push to Railway PostgreSQL → Railway serves dashboard only (AP never reachable from cloud)
5. **v3 Simulator tie-in** — Phase 3 bridge + FederatedTwinAgent Python port of equations
6. **Degraded RF injection** — synthetic DataStore injection mode + iperf3 stress test protocol
7. Sprint 3: FederatedTwinAgent (δ_calibration + δ_progress, MC sensitivity oracle)
8. Sprint 4: GS Convergence Protocol + HITL loop

---

## Session 4 Additions — June 9, 2026

### Console Cable Assessment
- AP3000 console = 115200 baud, 8N1 (`screen /dev/tty.usbserial-BG01WC92 115200`)
- Console and SSH are **fully independent** — console session does NOT block SSH
- Console adds NO value to automated data collection (baud rate ~14 KB/s, exclusive port)
- Console = recovery tool: boot-time kernel, password recovery, management IP loss, kdebug
- Connected, confirmed working. SSH remains sole collector transport.

### 802.11 Parameter Reference — Built June 9
- `docs/dt_parameter_reference.html` — 12 sections + §10a (12 optimizer knobs)
- NYT white/Libre Baskerville, breadcrumb nav ← Home / Sprint EOD / current
- Every metric: 802.11 clause, formula, units, range, AP CLI source, tier badge
- Commits 53c094a + 8ff2abc + 8f8f681 on main; live at GitHub Pages

### Validated Digital Twin Loop — June 9 Dialogue

**The 60/15/12 Split (confirmed in session):**
- 60 simulator params total
- 15 populated dynamically from AP SSH (the "bridge")
- 12 are optimizer knobs (tuned by OptimizerAgent)
- 45 are static DPM inputs (site geometry, client types, apps)
- 15 and 12 OVERLAP but are distinct sets

**QoE_t1 / QoE_t2 Iteration Flow:**
```
Iteration i at time t1:
  AP reports 15 measurements
  → SimulatorAgent(15 dynamic + 45 static) → QoE_t1 (5D predicted)

OptimizerAgent proposes Δ12_knobs:
  → SimulatorAgent(adjusted knobs) → QoE_predicted_t2

[HUMAN GATE: approve push to AP?]
  → AP reports 15 measurements at time t2
  → QoE_actual_t2

δ_progress    = QoE_actual_t2 − QoE_actual_t1       (goal progress)
δ_calibration = QoE_predicted_t2 − QoE_actual_t2    (model accuracy)

Iterate until: |δ_progress| < epsilon OR max_iterations
```

**5D QoE Vector (user-set epsilon per dimension):**
1. Throughput = avg Tx Mbps / client
2. Reliability = 1 − CRC%
3. Latency proxy = M/D/1: qd = 6u/(1−u); u = total CU fraction
4. Connection Quality = avg SNR → MCS tier
5. Capacity Headroom = 1 − (TxCU + RxCU)/100

**CONVERGENCE_CONFIG:**
- max_iterations=5, progress_threshold=10%, min_improvement=5%
- calibration_alarm=20%, mode=HITL

### Calibration Math — Gradient-Informed Inverse Calibration

For each QoE dimension d:
1. δ_d = QoE_actual_d − QoE_predicted_d (calibration error)
2. MC oracle: sensitivity[i] = ∂QoE_d/∂θ_i (perturb θ_i ±5%, measure QoE Δ)
3. i* = argmax(|sensitivity[i]|) → highest-leverage physics param
4. Δθ_i* = α × δ_d / sensitivity[i*] (α ≈ 0.1, gradient step)
5. Update θ_i* in simulator → reduces δ_calibration next iteration

MC oracle already exists in v3 simulator JS. Python port needed for CalibrationAgent.

### "Prius Display" — FederatedDashboard Concept (Sprint 4)

User-requested federated flow visualization. Like Toyota Prius hybrid display: components + flow between them, each clickable to drill down.

```
[AP3000] → [BRIDGE] → [SIMULATOR] → [QoE 5D]
   ↑                       ↑
   └─────── Human (HITL) ──┘
```

Click behavior:
- **AP3000**: opens existing live AP dashboard (localhost:8050)
- **BRIDGE**: 15-param table for current iteration (t1 vs t2 values + Δ per param)
- **SIMULATOR**: MC params + all 12 knob states + OPT_SEARCH coordinate ascent progress
- **QoE 5D**: convergence history (QoE_actual vs QoE_predicted, all 5 dimensions, all iterations)

Implementation: Sprint 4. Pure HTML+JS, Flask route `/federated`, data from JSON endpoints on Dash server. Sprint 4 AFTER local E2E validated.

### E2E Debug Plan — federated_debug.py (Sprint 2c, before Railway)

Single local script to validate all engines without Railway complexity:
1. Poll AP once via SSH (or `--replay session.csv` for offline testing)
2. Extract 15 bridge params → print bridge table
3. Run Python SimulatorAgent → compute QoE_t1 → print 5D vector
4. Run MC oracle → compute sensitivity_vector for all dimensions
5. Run coordinate ascent → print GS_1 recommendation (proposed Δknobs)
6. (Optional) apply proposed knobs to simulator → compute QoE_predicted_t2
7. Print all intermediate values with iteration # and δ_progress/δ_calibration

Enables full unit testing of optimizer with synthetic scenarios before adding PG/Railway.

### Agent Architecture — Control + Data Plane (June 9)

**APCollectorAgent** (built Sprint 1):
- CP: SSH poll every 5s (Paramiko), 5-command CLI sequence, adaptive retry
- DP: RadioMetrics + ClientMetrics → DataStore (in-memory), CSV archive, SQLite (Sprint 2a)

**SimulatorAgent** (Sprint 3):
- CP: receives 15-param bridge snapshot + CONVERGENCE_CONFIG → runs Python physics equations
- DP: outputs 5D QoE vector, MC sensitivity oracle via ±5% perturbation, QoE time series

**CalibrationAgent** (Sprint 3):
- CP: receives δ_calibration per dimension → calls MC oracle → gradient step
- DP: reads/writes simulator physics params θ_i, fires calibration_alarm if δ > 20%

**OptimizerAgent** (Sprint 3):
- CP: receives QoE_t1 + sensitivity_vector → coordinate ascent on OPT_SEARCH ranges
- DP: outputs GS_N recommendation (12 proposed Δknobs), convergence verdict

**FederatedTwinAgent** (Sprint 3, orchestrates all above):
- CP: CONVERGENCE_CONFIG state machine, manages iteration counter, owns HITL gate
- DP: GS History ledger, δ_progress + δ_calibration time series, feeds Prius display

**FederatedDashboard** (Sprint 4):
- CP: Flask route /federated + JSON endpoints per component
- DP: flow diagram state + per-iteration bridge snapshots + QoE convergence history

### Revised Sprint Map (June 9)

| Sprint | Phase | Focus | Prerequisite |
|--------|-------|-------|-------------|
| 2a | Foundation | storage.py SQLiteStore + session HTML | Start here |
| 2b | Knobs panel | knob_panel.py + dash-daq 4×3 grid | 2a done |
| 2c | E2E debug | federated_debug.py + synthetic injection | 2b done |
| 3 | Python agents | SimulatorAgent + CalibrationAgent + OptimizerAgent + FederatedTwinAgent | 2c validated |
| 4 | Prius display | FederatedDashboard /federated route, clickable flow | Sprint 3 done |
| 5 | Railway deploy | PG-backed DataStore, colleague access, no AP exposure | Sprint 4 validated |

Key change from June 5 plan: Sprint 2c (E2E debug) inserted BEFORE Railway. Validates all engines locally first. Railway is last, not second.

---

## Session 5 (continued) — Architecture Deep-Dive Decisions, June 9 2026

### R Decisions — ALL RESOLVED

| Decision | Answer |
|----------|--------|
| R1 — DB file | **Single persistent `digital_twin.db`** — enables cross-session calibration history queries in Sprint 2c |
| R2 — radio_polls fields | **All ⭐ fields added** — 31 existing + 33 recommended = **64 SQL fields** in radio_polls |
| R3 — knob_change_events | **Table created empty in Sprint 2a**, first producer wired in Sprint 2b (knob panel) |
| R4 — Session boundary | **One main.py run = one session** — SSH reconnects stay inside same session (Option A) |

### Final radio_polls Schema — 64 Fields (+ id, session_id, ts, radio, band = 69 columns total)

**Added ⭐ fields beyond the original 31:**
- Running averages (4): avg_tx_cu_pct, avg_rx_cu_pct, avg_interference_cu_pct, avg_noise_dbm
- Hardware: beacon_interval_ms (DTIM proxy knob)
- WiFi 6 flags (8): ofdma_dl, ofdma_ul, mu_mimo, bss_color, twt, beamforming, dynamic_chan_width, acsp_state
- EDCA: wmm_cw_max_be (CWmax ceiling)
- ACSP thresholds (5): interference_switch_pct, crc_switch_pct, power_floor_dbm, lb_airtime_limit_pct, dcw_trigger_threshold
- Mode flags (4): high_density, band_steering_enabled, load_balance_enabled, safety_net_enabled
- BGSCAN (3): bgscan_count, bgscan_missed, radar_count
- Airtime pct (2): rx_airtime_pct, tx_airtime_pct
- Short-term 10s rolling (4): st_tx_cu_pct, st_rx_cu_pct, st_int_cu_pct, st_noise_dbm
- Retries: tx_retry_pct

CSV-only (raw cumulative counters — NOT in SQL): rx/tx_packets_total, rx/tx_pkt_errors, rx/tx_pkt_dropped, rx/tx/crc_airtime_sec, tx_bytes_total, rx_bytes_total, tx_retry_count, tx_failed_count, tx_failed_pct (always None on AP3000)

### Three Planes — Confirmed Architecture

**Data Plane** (what flows right now):
AP3000 →SSH→ APCollectorAgent → DataStore(memory) + CSV + SQLite → Dash Dashboard
Sprint 5 adds: → Railway PostgreSQL → remote Dash read-only

**Control Plane** (what should change):
FederatedTwinAgent state machine → HITL gate → OptimizerAgent → CalibrationAgent
CONVERGENCE_CONFIG: max_iter=5, progress_thr=10%, min_improvement=5%, calib_alarm=20%, HITL

**Management Plane** (is the system healthy):
Phase 1 (Sprint 2a): /health endpoint (3 lines — status, last_poll_ts, session_id, uptime_s)
Phase 2 (Sprint Mgmt, before Railway): /config read API, Bearer token auth on dashboard, multi-AP inventory, watchdog restart, alert hooks

### ICDs — All 8 Defined, One Dedicated Sprint

ICD-01: AP ↔ Collector (HiveOS SSH CLI) — EXISTS in dt_parameter_reference.html §10
ICD-02: Collector ↔ DataStore (Python dataclass, in-process) — docstring sufficient
ICD-03: Collector ↔ SQLiteStore (Python API write path) — docstring sufficient
ICD-04: SQLiteStore ↔ report.py (Python API read path) — docstring sufficient
ICD-05: Backend ↔ Browser (Dash HTTP + WebSocket) — Dash framework, N/A
ICD-06: Federated JSON API (FederatedTwinAgent ↔ Browser) — MUST define before Sprint 4

ICD-06 spec (locked):
  GET /health         → { status: ok|degraded|down, last_poll_ts, session_id, collector_uptime_s }
  GET /api/twin-state → { state, iteration, session_id, last_poll_ts }
  GET /api/bridge     → { params: [{name, t1, t2, delta, tier}×15] }
  GET /api/simulator  → { physics_params: {}, knob_states: {12}, mc_log: [] }
  GET /api/qoe-history→ { iterations: [{i, qoe_actual_5d, qoe_predicted_5d, delta_progress, delta_calibration}] }

ICD-07: Collector ↔ Railway PG (TCP/libpq) — define before Sprint 5
ICD-08: SimulatorAgent ↔ CalibrationAgent (SensitivityMatrix dataclass) — define before Sprint 3

Sprint ICD deliverable: formal HTML reference doc (like dt_parameter_reference.html) with all 8 ICDs.
Timing: Sprint ICD runs BETWEEN Sprint 3 and Sprint 4 (ICD-06 gates Sprint 4 coding).

### Updated Sprint Map — FINAL (June 9 2026)

| Sprint | Deliverable | Key decisions |
|--------|-------------|---------------|
| 2a | storage.py (64-field schema) + report.py + /health | Single DB, WAL, 64 fields, /health 3 lines |
| 2b | knob_panel.py + dash-daq 4×3 grid + knob_change_events producer | All 12 knobs rendered |
| 2c | federated_debug.py + synthetic injection (--inject scenario.json) | Full loop local-only |
| 3 | SimulatorAgent + CalibrationAgent + OptimizerAgent + FederatedTwinAgent | ICD-08 defines interface |
| ICD | All 8 ICDs as formal HTML doc (icd_reference.html) | ICD-06 gates Sprint 4 |
| 4 | Prius FederatedDashboard + /api/* endpoints | ICD-06 spec must be approved |
| Mgmt | /config API + Bearer token auth + multi-AP inventory + watchdog | Before Railway expose |
| 5 | Railway deploy — PG-backed DataStore, remote read-only dashboard | ICD-07 defines PG interface |

### What's NOT in SQL (confirmed CSV-only)
Raw cumulative counters (derive in analysis): rx/tx_packets_total, rx/tx_pkt_errors,
rx/tx_pkt_dropped, rx/tx/crc_airtime_sec, tx_bytes_total, rx_bytes_total,
tx_retry_count, tx_failed_count, tx_failed_pct (always None on AP3000),
snap_tx/rx/int_cu_pct (noisier than st_ averages), spectral_scan (debug only),
radio_mac (constant — store once in sessions table), tx_failed_pct (never populated)

### SQLite Schema Version Guard (to avoid migration pain)
Add schema_version table in Sprint 2a:
  CREATE TABLE schema_version (version INTEGER NOT NULL);
  INSERT INTO schema_version VALUES (1);
Future column additions: check version on open, run ALTER TABLE migrations in order.
