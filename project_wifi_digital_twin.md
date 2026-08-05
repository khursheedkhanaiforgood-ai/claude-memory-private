---
name: WiFi Digital Twin Platform — Full Architecture
description: E2E collaborative WiFi design-to-deployment-to-optimization platform. Digital twin with predictive design, live EP1 telemetry mirror, Broadcom RRM simulation, XIQ config tuning. Teaching agent bridges 5G↔WiFi throughout.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Vision
A multi-tenant, real-time collaborative WiFi Digital Twin where globally distributed teams co-design, simulate, configure, deploy, and continuously optimize enterprise WiFi — guided by AI agents validated against the 802.11be standard and private WLPC corpus.

**It is a true digital twin:**
- **Pre-deployment**: Predictive model — capacity, link budget, channel plan, config
- **Post-deployment**: Live mirror — EP1 telemetry feeds the same simulation engines
- **Continuous**: RRM simulation runs in parallel with real APs, predicting AutoRF convergence
- **Feedback loop**: Predicted vs actual gap → Tuning Agent → XIQ config delta → push

**Why:** User comes from 5G/cellular (not WiFi domain). Teaching Agent bridges 5G↔WiFi at every step.

---

## E2E Lifecycle (3 Loops)

```
┌─────────────────────────────────────────────────────────────────────┐
│                    DESIGN LOOP (pre-deployment)                     │
│  Requirements → Capacity → Link Budget → AP Selection → Channel     │
│  Planning → Config Generation → XIQ Push → Switch CLI Push          │
└─────────────────────────────┬───────────────────────────────────────┘
                              ↓  DEPLOY
┌─────────────────────────────────────────────────────────────────────┐
│               OPTIMIZATION LOOP (continuous post-deploy)            │
│  EP1 Telemetry → RRM Simulator → Predicted vs Actual Gap →          │
│  Tuning Agent → Config Delta → XIQ API Push                         │
│                                                                     │
│  RRM Simulation mirrors Broadcom AutoRF:                            │
│  DCS (channel) + TPC (power) + BSS Color + Multi-AP Coord (WiFi 7)  │
└─────────────────────────────┬───────────────────────────────────────┘
                              ↓  EVENT
┌─────────────────────────────────────────────────────────────────────┐
│                TROUBLESHOOTING LOOP (event-driven)                  │
│  EP1 Alert / User Symptom → 30-Row Diagnosis → Fix Recommendation   │
│  → Config Delta → XIQ Push / CLI Fix                                │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Full Agent Topology

```
ORCHESTRATOR (Lead — state machine for all 3 loops)
│
├── INTAKE AGENT
│   └── Socratic requirements → SiteRequirements JSON
│       Teaching: client mix = UE distribution, SLA = QoS bearer
│
├── DESIGN AGENT
│   ├── Capacity Planning Sub-agent (Rev Wi-Fi airtime model + OFDMA)
│   ├── Link Budget Sub-agent (SNR→MCS, 802.11b→802.11be MCS 0-13)
│   ├── AP Selection Sub-agent (AP3000/4000/5020/5022/5060, PoE check)
│   ├── Channel Planning Sub-agent (2.4/5/6 GHz, DFS, PSC, CCI score)
│   └── RAG Validation Sub-agent (corpus + 802.11be-2024 + web)
│
├── CONFIG AGENT
│   ├── XIQ Config Sub-agent → Device Template JSON
│   │   (SSID profiles, radio settings, QoS, 802.11r/k/v, OFDMA)
│   │   Dual output: static profile + AutoRF profile (with caveats)
│   ├── Switch Config Sub-agent → EXOS CLI + VOSS CLI (separate scripts)
│   │   (VLAN, PoE, trunk/access, FA I-SID mapping, VLAN↔SSID binding)
│   └── Qualification Sub-agent (WLANPros Not-Wireless pre/post checklist)
│
├── RRM SIMULATION AGENT  ← THE DYNAMIC LAYER
│   │  Runs continuously. Mirrors what Broadcom AutoRF does on real APs.
│   │  Fed by: DesignSession (pre-deploy) + EP1 telemetry (post-deploy)
│   │
│   ├── DCS Simulator
│   │   Input: neighbor AP RSSI scan, channel utilization per channel
│   │   Logic: replicates Broadcom channel selection algorithm
│   │   Output: predicted channel assignment per AP after convergence
│   │   Validates: does AutoRF converge to our designed channel plan?
│   │
│   ├── TPC Simulator
│   │   Input: neighbor AP signal levels, coverage requirements
│   │   Logic: TX power step-down algorithm (WLANPros target: 7/13/16 dBm)
│   │   Output: predicted TX power per radio after convergence
│   │   Validates: no coverage holes from over-aggressive power reduction
│   │
│   ├── BSS Color Simulator (802.11ax)
│   │   Input: neighbor AP scan, current BSS color assignments
│   │   Logic: 802.11ax color conflict detection + re-assignment
│   │   Output: BSS color map across all APs
│   │   Effect: reduces unnecessary deferrals between non-overlapping BSSs
│   │
│   └── Multi-AP Coordination Simulator (WiFi 7, AP5020/5060 only)
│       Input: AP5020/5060 neighbor topology, client SNR measurements
│       Logic: mirrors 802.11be Multi-AP Coordination (≈ CoMP in 5G)
│       Output: coordinated null-steering + spatial reuse schedule
│       Teaching: Multi-AP Coord ≈ CoMP; BSS Color ≈ ICIC
│
├── EP1 INTEGRATION AGENT  ← OBSERVABILITY LAYER
│   Input: Extreme Platform One telemetry stream
│   Ingests: channel utilization, RSSI per client, SNR, retry rates,
│            roaming events (802.11r/k/v), client counts, AP health,
│            Broadcom chipset diagnostics (BSS load, airtime stats)
│   Outputs:
│   ├── Feed to RRM Simulation Agent (live environment inputs)
│   ├── Feed to Tuning Agent (predicted vs actual comparison)
│   └── Feed to Troubleshooting Agent (alert triggers)
│
├── TUNING AGENT  ← OPTIMIZATION FEEDBACK LOOP
│   Input: RRM Simulator predictions + EP1 actual measurements
│   Process:
│   ├── Gap Analysis: predicted channel/power/utilization vs actual
│   ├── Root Cause: which Broadcom parameter needs adjustment?
│   │   (MBR, RX-SOP, association limit, channel width, BSS Color,
│   │    OFDMA on/off, TWT intervals, beamforming, 4K-QAM threshold)
│   └── Config Delta: generates XIQ template update + human explanation
│   Output: XIQ API call (config delta) + Teaching Agent commentary
│   Teaching: tuning loop ≈ SON MRO (Mobility Robustness Optimization)
│
├── TROUBLESHOOTING AGENT
│   ├── L1→L7 Diagnosis Sub-agent (30-row WLANPros causes table)
│   │   WIRELESS (rows 1-15) → LOCAL NETWORK (16-28) → INTERNET (29-30)
│   ├── Spectrum Analysis Sub-agent (13 interferer signatures from corpus)
│   ├── Debug Commands Sub-agent (EXOS + VOSS + XIQ CLI)
│   └── WLANPi Sub-agent (modes, commands, active survey)
│
├── TEACHING AGENT  ← CROSS-CUTS EVERY AGENT
│   Mode 1: Explain (learner receives concept explanation)
│   Mode 2: Socratic (ask before reveal — Socratic method)
│   Mode 3: Bridge (5G↔WiFi — PRIMARY mode for this user)
│   Mode 4: Cite (802.11be section reference + corpus citation)
│   Always called after each major agent output
│
└── RAG VALIDATION AGENT  ← CROSS-CUTS EVERY AGENT
    Layer 1: Private corpus query (T1 > T2 > T3)
    Layer 2: 802.11be-2024 section lookup (AUTHORITATIVE — wins on conflict)
    Layer 3: Web search (post-2024 errata, vendor advisories)
    Output: {claim, corpus_verdict, standard_verdict, web_verdict,
             final_verdict, citations, conflict_flag}
```

---

## RRM Dynamic Layer — Detail

### Why this is the most important layer for a digital twin

In 5G, SON runs network-wide with centralized visibility (O-RAN RIC, vendor SON). In WiFi, every AP runs RRM independently via CSMA/CA — there is no central scheduler. This creates emergent behavior that is hard to predict. The RRM Simulation Agent exists specifically to:

1. **Predict convergence** — given our designed channel/power plan, what will AutoRF settle on after 24-48 hours of operation?
2. **Detect oscillation** — will DCS cause channels to flip back and forth (known failure mode)?
3. **Validate against design intent** — if AutoRF converges to a different channel plan than designed, is the new plan better or worse?
4. **Pre-tune thresholds** — CCI threshold (>-85 dBm per WLANPros), retry rate trigger, load threshold — these feed into the DCS algorithm

### Broadcom AutoRF inputs (from chipset diagnostics via EP1)
- Per-channel noise floor (measured on idle radio)
- Neighbor AP RSSI per channel (from background scanning)
- Channel utilization % (Tx + Rx + busy)
- Retry rate per radio
- BSS Load IE (client count + channel utilization)
- CRC error rate
- Roaming event frequency

### RRM simulation outputs
- Predicted channel per AP (post-convergence)
- Predicted TX power per AP radio
- Predicted channel utilization (with new channel plan)
- CCI risk map (overlay of predicted assignments)
- Oscillation risk flag (if DCS thresholds too close to trigger)

---

## Broadcom Chipset Parameter Map

### CONFIGURABLE via XIQ (user tunes these)
| Parameter | XIQ Setting | RRM Role |
|-----------|-------------|----------|
| TX Power (min/max) | dBm slider | TPC bounds |
| Channel | Manual or DCS | DCS seed |
| Channel Width | 20/40/80/160/320 MHz | DCS + capacity |
| OFDMA | Enable/disable UL+DL | Airtime efficiency |
| MBR (Min Basic Rate) | 12/24 Mbps | Cell size control |
| RX-SOP | dBm threshold | CCI rejection (high-density) |
| Association Limit | Max clients/radio | Load threshold for LB |
| 802.11r | FT enable | Roaming optimization |
| SDR band mode | Dual-5 or dual-6 (AP5020/5060) | Spectrum allocation |
| Preamble Puncturing | Enable/disable | 802.11be WiFi 7 |
| Multi-AP Coordination | Enable/disable | WiFi 7 CoMP |
| AutoRF CCI threshold | dBm | DCS trigger |
| AutoRF retry threshold | % | Channel change trigger |

### FIXED / CHIPSET-MANAGED (Broadcom — not user-tunable)
| Parameter | Behavior | Observable via EP1 |
|-----------|----------|-------------------|
| Spatial streams | Hardware-fixed (2×2 AP3000, 4×4 AP5000 series) | PHY rate reports |
| MCS index selection | Auto per SNR | Per-client MCS stats |
| 4K-QAM activation | Auto when SNR ≥ ~36 dB | MCS 12/13 usage % |
| TWT interval scheduling | Negotiated per client | TWT session logs |
| OFDMA RU sizes | Auto-allocated | RU utilization stats |
| BSS Color value | Auto (1–63) | Color conflict events |
| HARQ retransmission | WiFi 7, chipset only | Retry reduction stats |
| MLO link preference | AP+client negotiation | MLO link stats |
| Beamforming weights | Per-client, per-packet | Beam SNR improvement |

---

## EP1 Integration Architecture

```
Deployed APs + Switches
        │  (telemetry)
        ▼
Extreme Platform One (EP1)
  - Network visibility
  - AI/ML analytics
  - Performance telemetry
  - Alert engine
        │
        │  EP1 API
        ▼
EP1 Integration Agent
  ├── Telemetry normalizer (EP1 format → DesignSession metrics)
  ├── Alert parser (EP1 alerts → Troubleshooting Agent triggers)
  └── Telemetry store (TimescaleDB or InfluxDB time-series)
        │
        ├──→ RRM Simulation Agent (live RF environment inputs)
        ├──→ Tuning Agent (predicted vs actual comparison)
        └──→ Troubleshooting Agent (symptom data)
```

---

## Deployment Architecture

```
RAILWAY (shared platform)
├── FastAPI backend (WebSocket + REST)
├── PostgreSQL (sessions, users, design history, config packages)
├── Redis (real-time presence, session state)
├── Agent orchestration services
└── EP1 Integration Agent (receives EP1 webhook/API data)

LOCAL (each user's machine — private corpus stays here)
├── ~/.wifi-rag/corpus/         PRIVATE — never committed
├── ~/.wifi-rag/embeddings/     ChromaDB vector store
└── RAG local API (localhost:8001) — Railway backend queries this

EXTREME NETWORKS INFRASTRUCTURE
├── ExtremeCloud IQ (XIQ) — config push target
├── Extreme Platform One (EP1) — telemetry source
└── APs + Switches — physical deployment target

GITHUB (public)
└── wifi-mastery/ source code — NO corpus files
```

---

## 5G→WiFi Bridge (Teaching Agent Core)

| WiFi Concept | 5G Equivalent | Key Difference |
|-------------|---------------|----------------|
| MLO (WiFi 7) | Carrier Aggregation | WiFi MLO is AP+client negotiated; CA is network-assigned |
| TX Power slider | P-Max | WiFi has no UE power control feedback loop |
| MCS 0–13 | NR MCS 0–28 | WiFi MCS 13 = 4K-QAM (WiFi 7 only, SNR ≥36 dB) |
| HARQ (WiFi 7) | HARQ | Both chipset-managed; WiFi HARQ is newer/less mature |
| TWT | eDRX | TWT is per-device negotiated; eDRX is network-paged |
| OFDMA scheduler | gNB scheduler | WiFi OFDMA layered ON TOP of CSMA/CA; not replacing it |
| WMM AC (VO/VI/BE/BK) | DRB QoS class | WiFi has 4 ACs; NR has many more bearer types |
| RSSI/SNR | RSRP/RSRQ | Same physics; different measurement definitions |
| XIQ RRM AutoRF | SON (ANR+CCO+MLB) | SON is centralized+predictive; AutoRF is per-AP+reactive |
| 802.11r FT | X2 handover | WiFi FT is client-initiated; X2 is network-initiated |
| BSS Coloring | ICIC | Both reduce inter-cell interference; different mechanisms |
| Multi-AP Coord | CoMP | Both coordinate multi-cell transmission; WiFi is newer |
| Airtime % | PRB utilization % | Same concept: shared medium utilization |

**Critical 5G insight:** Only 25% of WiFi frames carry user data (vs >90% in 5G). 80% of all WiFi frames are <256 bytes. This is why OFDMA is so impactful — it packs multiple small frames into parallel Resource Units.

---

## Build Sequence

| Block | Sprint | Description |
|-------|--------|-------------|
| 1: RAG | Sprint 1 | Ingest pipeline, ChromaDB, local RAG API |
| 2: Core Agents | Sprint 1 | Orchestrator, Teaching, RAG Validator, Design, Config, Intake |
| 3: Computation Engines | Sprint 1 | Link Budget, Airtime, AP Selection, Channel Planning |
| 4: API + Backend | Sprint 1 | FastAPI, WebSocket, PostgreSQL, Redis, Railway |
| 5: RRM Simulation | Sprint 2 | DCS/TPC/BSS Color/Multi-AP simulators |
| 6: EP1 Integration | Sprint 2 | Telemetry ingestion, Tuning Agent, feedback loop |
| 7: Hardware Push | Sprint 2 | XIQ API, Netmiko SSH (EXOS + VOSS) |
| 8: Troubleshooting | Sprint 2 | 30-row agent, spectrum, debug commands, WLANPi |
| 9: Collab Frontend | Sprint 2 | React, role-based views, Teaching sidebar |
| 10: Standards Index | Sprint 3 | Full 802.11be-2024 + Broadcom param map |
| 11: EVE-NG Bridge | Sprint 3 | Lab simulation integration |
