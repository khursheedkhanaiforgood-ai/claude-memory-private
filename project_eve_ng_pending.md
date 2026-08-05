---
name: EVE-NG Lab Platform — Pending Features & Branch Structure
description: Active branches, pending features, and architecture notes for eve-ng-lab-platform
type: project
---

## Branch Structure
- `main` — LOCKED/PROTECTED stable version. Do not push without user authorization (password = year of older daughter's birth).
- `feature/traceroute` — Traceroute feature branch (separate from main, user decides when to merge)

## Completed (in main)
- WebSocket agent, config extraction, ping, push commands, run cancel/stop
- Topology map with colored links, node shapes, export as PNG/SVG
- Export TXT + Export CSV (Excel-compatible structured output)
- exec-timeout pushed to Core-1 and Site_1-R1 (devices won't go idle)
- Remaining devices needing exec-timeout: Site_1-R2, Site_2-R1, Site_2-R2, Site_1-SW1

## Pending / Deferred
1. **NSSM Windows Service** — Install agent as Windows Service (auto-start/restart). Command scaffold ready.
2. **EVE-NG folder/lab browser** — Dynamically discover EVE-NG folder structure via REST API and display in UI.
3. **Push commands double-send fix** — Auto-appended `end` + `write memory` causes double-send; user must omit these from input for now.
4. **Windows Agent remote access** — User wants to run/manage the Windows agent remotely (RDP or similar).

**Why:** User wants less manual config, self-service UI, and stable production environment.
**How to apply:** Raise pending items at the start of next session. Always confirm before touching main.
