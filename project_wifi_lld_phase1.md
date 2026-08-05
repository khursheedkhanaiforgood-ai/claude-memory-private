---
name: WiFi Digital Twin — LLD Phase 1 Status
description: LLD Phase 1 COMPLETE (Apr 29 2026). 9 modules + Appendix A. XIQ config ref = 110 params with Override badge. WiFi7Params expanded. Ready for Sprint 1 coding.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Phase 1 LLD Scope
Uniform AP generation per site. No mixed mode. No floor plan. User provides RF coverage area estimate.

## LLD file locations
- Markdown: `/private/tmp/wifi-mastery/LLD-phase1-wifi-digital-twin.md`
- HTML: `/private/tmp/wifi-mastery/docs/lld-phase1-wifi-digital-twin.html`
- Live: https://khursheedkhanaiforgood-ai.github.io/wifi-mastery/docs/lld-phase1-wifi-digital-twin.html
- XIQ param table direct: https://khursheedkhanaiforgood-ai.github.io/wifi-mastery/docs/lld-phase1-wifi-digital-twin.html#dpm-ref

## 9 LLD Modules

| Module | Layer | Description |
|--------|-------|-------------|
| L1 | Data Models | Pydantic v2 schemas for all DPM entities (Sections A–H) |
| L2 | Link Budget Engine | SNR→MCS, WiFi6LinkBudget (MCS 0–11), WiFi7LinkBudget (MCS 0–13), ComparisonEngine |
| L3 | Airtime Calculator | Rev Wi-Fi formula + OFDMA ×1.30 modifier + VoIP trap |
| L4 | AP Selection Engine | Model filter, PoE budget check, SDR flag |
| L5 | Channel Planning Engine | 2.4/5/6 GHz, DFS, PSC, CCI scoring (reuse ratio) |
| L6 | Intake Agent | Socratic→DPM, Sections A–G (+ H for WiFi 7) |
| L7 | Design Agent + Orchestrator | Iterative capacity↔RF loop, 12-state FSM, WebSocket |
| L8 | Config Agent | XIQ template YAML, EXOS CLI, VOSS CLI |
| L9 | API Layer | FastAPI endpoints, WebSocket, data flows |

## Appendix A: Mixed-Mode Input Parameters
12 additional DPM params for future mixed-mode LLD (Sprint 4+). Includes adjacency graph, per-zone AP generation, zone border CCI, MLO roam handling. Saved inside LLD — does NOT overlap Phase 1 design.

## XIQ Config Reference Table (110 params) — committed Apr 29 Session 2
- 6 sections: Radio (29), SSID (30), AutoRF (15), QoS (12), WIDS (12), System (12)
- **`channel_width_6g`** added as param #4 (W6+, Override) — was missing from prior table
- **Source badge taxonomy** (5 types):
  - `src-u` Blue = User mandatory (Socratic blocks)
  - `src-ov` Teal = Override — safe default, user should know + can change, safe range shown
  - `src-def` Amber = System default, rarely changed
  - `src-f` Red = Fixed (chipset/standard locked)
  - `src-l8` Dark gray = Config Agent hard-codes
- **25 Override params** each have safe operating range text in the Default column
- Landing page quick-link: "▶ 110-Param XIQ Reference" → `#dpm-ref`

## WiFi7Params (L1 Pydantic model) — Apr 29 Session 2
Two fields added:
```python
multi_ru_enabled:  bool = True   # Multi-RU per client (802.11be)
restricted_twt:    bool = False  # deterministic latency for industrial IoT
```

## Status: COMPLETE ✓
- [x] LLD document created
- [x] Data models detailed (Pydantic v2, all DPM sections A–H)
- [x] Engine specs written (L2–L5 with full formulas)
- [x] Agent flows diagrammed (L6–L9, 12-state FSM)
- [x] Mixed-mode appendix written (Appendix A)
- [x] 110-param XIQ config reference committed + live

## Next: Sprint 1 Coding (Day 1)
```bash
cd /private/tmp/wifi-mastery
mkdir -p src/{models,engines,agents,api,rag}
touch src/__init__.py src/models/dpm.py src/engines/link_budget.py src/engines/airtime.py
```
