---
name: Simulator font audit deferred to June 4
description: Font sizes throughout simulator v2 are too small — full audit sprint planned for June 4 Sprint 2
type: feedback
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
All inline font-size overrides in stadium_wifi_simulator_v2.html are too small for a 1440p display. User flagged this explicitly June 3.

Defer font audit to Sprint 2 (June 4 late morning) — do NOT attempt piecemeal fixes before physics (Sprint 1) is done.

**Why:** Font changes are cosmetic and will touch 40+ inline style overrides. Doing them before the physics rewrite means touching the file twice. Bundle with Sprint 2 structural changes.

**How to apply:** In Sprint 2 (after C1/C2/C3 physics fixes), do a single-pass font audit:
- All `font-size:8px` → `font-size:11px`
- All `font-size:9px` → `font-size:11px` (labels, tab text, descriptions)
- All `.kd` descriptions → `font-size:11px`
- `.bpft`, `.ralt`, `.mn`, `.mu` → `font-size:11px`
- Summary label `font-size:7px` in modal → `font-size:10px`
