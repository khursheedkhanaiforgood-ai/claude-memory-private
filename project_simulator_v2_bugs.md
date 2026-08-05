---
name: Simulator v2 Bug Report — June 3, 2026
description: 17 physics bugs found in stadium_wifi_simulator_v2.html; sprint plan for June 4
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
Simulator v2 physics model is fundamentally broken — 17 bugs found via full audit June 3 2026.

**Why:** Multiplier chain stacks 12.9× over real PHY rate → capacity 46,000 Gbps (should be ~2,000 Gbps) → airtime 2% (should be 20–60%) → all outputs meaningless.

**Bug tracker + verbatim dialogue:** `docs/simulator_v2_addendum.html` (live on GitHub Pages)
**Simulator:** `docs/stadium_wifi_simulator_v2.html`
**Repo:** `5320-onboarding-agent` at khursheedkhanaiforgood-ai/5320-onboarding

## Root causes (CRITICAL — fix first)
- C1: `effPHY = phy * muF` — MU-MIMO 2.8× applied ON TOP of PHY that already has 4SS
- C2: `capMult()` stacks 14 multipliers to 4.62× uncapped — should max at ~2.0×
- C3: airtime `u` uses inflated capacity; goodput `gp` uses raw PHY — contradictory

## Tomorrow's Sprint Plan (June 4)

**Sprint 1 — Morning — CRITICAL (C1, C2, C3, H1, L1)**
- Add AP Count slider (default 280, range 50–500)
- Remove muF from effCap; apply per-band based on actual antenna count
- Cap capMult() at 2.0× saturation
- Make gp and u share same capacity denominator
- Expected result: airtime 20–60%, capGbps 1,400–3,500 Gbps

**Sprint 2 — Late Morning — HIGH + fonts (H2, H3, H4, H5, L3)**
- Wire OFDMA UL to actual computation
- Normalize band splits after enhRNR boost
- Separate DL/UL airtime (half-duplex model)
- Fix snrMod.g6: +4 → −6 (outdoor path loss)
- Fix "of N total" clients display
- Font audit: all 8-9px → 11-12px

**Sprint 3 — Afternoon — refinements (H6, H7, M1, M2, M4)**
- Fix queueing delay: coefficient 6 → 1.5
- Fix MC label: Uniform not Poisson
- Fix TX power coefficient: 0.5 → 1.0
- Fix rTWT formula: SP interval not lat×0.12

**Sprint 4 — Future (M3, L2, Track B)**
- Degrade score to interact with physics
- GI 800ns: 5% → 11%
- Track B: validate against AP5060 telemetry

## Key finding from today's dialogue
User: "The outputs don't make sense... how can you calculate the airtime is only 2% and 287Gbps demand? How many APs did you take? I did not give any APs?"
→ 280 APs hardcoded at line 633 and 726. User never set it. Exposed in audit.

**How to apply:** On June 4 login, open addendum first, start Sprint 1 immediately.
