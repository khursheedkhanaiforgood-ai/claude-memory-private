---
name: WiFi Digital Twin — Sprint 1 Plan (3-Day)
description: Sprint 1 plan: 17 modules across 3 days. LLD + 110-param XIQ table fully committed Apr 29. Python coding NOT YET STARTED. Next session = Day 1. E2E test = convention center 929m² / 3K users.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Status as of Apr 29 2026 (Session 2 EOD)
- LLD Phase 1: ALL 9 modules fully specified + 110-param XIQ config ref committed
- HLD v1.1: complete, live at GitHub Pages
- Python coding: NOT STARTED — begin next session

## E2E Test Case (replaces generic office 500m²)
Convention center scenario — use for M16 integration test:
```
Site:        convention center, 10,000 sqft (929 m²)
Peak users:  3,000 (3.2 users/m² — sports arena tier density)
WiFi gen:    WiFi 7 (802.11be)
Apps:
  50% Zoom video  → 1,500 users × ~2.5–4 Mbps (quality TBD)
  30% VoIP G.711  → 900 users × 128 Kbps (no AMPDU — VoIP trap fires)
  20% Slack+YT    → 600 users × ~5 Mbps
Capacity dominates over coverage (capacity AP count >> coverage AP count)
OFDMA ×1.30 modifier triggers (>50% small frames from VoIP + Slack)
```
**DPM Socratic gaps still needed from user:**
- ceiling height (affects 6GHz free-space loss — 4m vs 10m = ~30% AP count difference)
- device mix (phone/tablet/laptop %)
- WiFi generation split of client fleet
- Zoom quality tier (720p=2.5 Mbps vs 1080p=4 Mbps — critical for load calc)
- YouTube resolution target
- SSID count + VLAN scheme
- DFS tolerance, 6GHz enabled
- WPA3-only or mixed
- AutoRF vs manual channel lock

## 3-Phase Navigation UX (agreed Apr 29 Session 2)
```
Phase 1: DESIGN INTAKE    [■■■□□□□] Socratic → DPM JSON (Sections A–H)
  Progress bar: X/7 sections complete
  Show what captured + why asking + engine implication
  4 blocker params gate Phase 2: ceiling height, Zoom quality, device mix, country code

Phase 2: DESIGN ENGINES   [□□□□□□□] Link Budget → Airtime → AP Count → Channel Plan
  Runs automatically after Phase 1 complete
  Shows intermediate results + Teaching Agent explanations

Phase 3: XIQ CONFIG       [□□□□□□□] Override review (25 params) → Config export
  User reviews/adjusts Override params before export
  Config Agent generates XIQ JSON + EXOS/VOSS CLI
```

## Day 1 Kick-off Command
```bash
cd /private/tmp/wifi-mastery
mkdir -p src/{models,engines,agents,api,rag}
touch src/__init__.py src/models/dpm.py src/engines/link_budget.py src/engines/airtime.py
```

## Key Code Contracts (from LLD)
```python
# Strategy Pattern — L2
class WiFi6LinkBudget(LinkBudgetStrategy):
    def max_mcs_index(self) -> int: return 11

class WiFi7LinkBudget(WiFi6LinkBudget):
    def max_mcs_index(self) -> int: return 13
    # upgrades to MCS 12/13 only when SNR >= 36 dB AND protocol == 802.11be

# Airtime formula — L3
airtime_pct = app_kbps / device_kbps  # per active client
available = rf_environment_pct - ssid_beacon_overhead_pct
ap_count = ceil(sum(active * airtime) / available)
# VoIP trap: G.711 = 128 Kbps (NOT 64) — 160-byte frames, no AMPDU
# OFDMA: × 1.30 when >50% frames < 256 bytes
```

## Sprint 1 Goal
Working E2E loop by end of Day 3: `requirements intake → capacity plan → link budget → AP count → config generation` with RAG validation + Teaching Agent at every step. CLI-testable. Railway-deployed backend.

## Day 1 — Foundation Layer

### Batch 1A (Parallel)
| Module | Deliverable |
|--------|------------|
| M1: RAG Ingest Pipeline | `~/.wifi-rag/ingest.py` — ChromaDB, tier weighting T1>T2>T3, 802.11be section-level chunks |
| M2: Core Data Models | `src/models/` — Pydantic v2: SiteRequirements, ClientDevice, Application, APModel, CapacityPlan, LinkBudget, DesignSession, ConfigPackage |
| M3: 802.11be Indexer | Part of ingest.py — metadata flag `source: IEEE_802.11be-2024` + section numbers preserved |

### Batch 1B (Sequential after 1A)
| Module | Deliverable |
|--------|------------|
| M4: Local RAG API | `src/rag/server.py` — FastAPI localhost:8001. Endpoints: POST /validate, POST /query, GET /health |
| M5: Link Budget Engine | `src/engines/link_budget.py` — full SNR→MCS: 802.11b/g/n/ac + 802.11ax MCS10/11 + 802.11be MCS12/13 (4K-QAM). Validated vs 802.11be-2024 Table 36-122 |

## Day 2 — Intelligence Layer

### Batch 2A (Parallel)
| Module | Deliverable |
|--------|------------|
| M6: Airtime Calculator | `src/engines/airtime.py` — Rev Wi-Fi model + OFDMA extension (802.11ax RU scheduling) |
| M7: AP Selection Engine | `src/engines/ap_selector.py` — budget filter → AP3000/4000/5020/5022/5060, PoE budget validator |
| M8: Channel Planning Engine | `src/engines/channel_planner.py` — 1/6/11 (2.4G), U-NII bands + DFS (5G), PSC + RNR (6G). CCI risk score |

### Batch 2B (Sequential after 2A)
| Module | Deliverable |
|--------|------------|
| M9: Teaching Agent | `src/agents/teaching_agent.py` — 4 modes: explain/socratic/bridge/cite. Bridge mode primary |
| M10: RAG Validation Agent | `src/agents/rag_validator.py` — 3-layer: corpus → 802.11be-2024 → web. Standard wins on conflict |
| M11: Intake Agent | `src/agents/intake_agent.py` — Socratic → SiteRequirements JSON |

## Day 3 — Config + Orchestration + Integration

### Batch 3A (Parallel)
| Module | Deliverable |
|--------|------------|
| M12: Config Agent | `src/agents/config_agent.py` — XIQ Device Template JSON + EXOS CLI + VOSS CLI + AutoRF config block (dual: static + AutoRF profiles) |
| M13: Design Agent | `src/agents/design_agent.py` — capacity → channel → RAG validation loop. Iterative Capacity↔RF |
| M14: FastAPI Backend | `src/api/main.py` — REST + WebSocket. PostgreSQL + Redis. Railway-ready |

### Batch 3B (Sequential after 3A)
| Module | Deliverable |
|--------|------------|
| M15: Orchestrator | `src/agents/orchestrator.py` — state machine: intake → design → config → validate → teach |
| M16: E2E Integration Test | `tests/test_e2e.py` — convention center 929m², 3K users, Zoom+VoIP+streaming, AP5060. |
| M17: Railway Deploy | `railway.toml` + `Dockerfile`. RAG stays local |

## Sprint 2+ Deferred
- React frontend + collaborative UI
- XIQ API push + Netmiko SSH
- RRM simulation (AutoRF convergence model)
- EP1 telemetry ingestion + Tuning Agent
- Multi-AP Coordination simulation (WiFi 7)
- Troubleshooting Agent
- Multi-user WebSocket
- EVE-NG lab integration
- Coverage design with floor plan (Phase 2 LLD)

## RRM Gap Note
AutoRF config block added to M12 (Config Agent) Day 3. Sprint 2 gets full RRM simulation module mirroring Broadcom AutoRF convergence, fed by EP1 telemetry. See project_wifi_digital_twin.md for full RRM architecture.
