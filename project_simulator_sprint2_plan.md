---
name: Simulator v3 Sprint 2 — Sequenced Implementation Plan
description: 18-item Sprint 2 plan for stadium_wifi_simulator_v3.html. Integrates strategy paper (Journey to Digital Twin PDF). Physics-grounded archetypes, regime confirmation panel, SLA central state, DL/UL split. June 4 2026.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
## Source Documents
- Strategy paper: `/Users/khukhan/Downloads/p_June4_Journey to the Digital Twin — Extreme AP : EP1 Parameter Twin.pdf`
- Simulator: `/Users/khukhan/5320-onboarding-agent/docs/stadium_wifi_simulator_v3.html`
- Branch: `feature/simulator-v3` (v3 also published to main for GitHub Pages)

## Reset Button Clarification (from planner audit)
Two distinct buttons currently in simulator header:
- `★ Stadium Optimal` → calls `applyOptimal()` → sets best-known stadium golden-set values
- `↺ Reset` → calls `resetAll()` → sets ALL params to **WORST CASE** (Gap Analysis baseline)
**Decision**: relabel both honestly: `★ Apply Golden Set` + `▼ Load Worst Case`. Both kept.

## Archetype Decision (from PDF §02)
Five physics-grounded archetypes per §02 Table V0 — NOT venue names:
1. Dense Public Venue (stadium/arena) — Capacity + fairness
2. High-Density Enclosed (lecture hall/office) — Capacity + stability
3. Sparse High-Throughput (warehouse/plant) — Coverage + roaming
4. Mixed Retail/Hospitality (store/hotel) — POS SLA + guest seg.
5. Latency-Critical Low-Density (medical/control) — Jitter/latency

Convention Center = sub-class of Dense Public Venue (per §02 footnote). Not a standalone archetype until §05 driver discovery confirms it.

## Key Design: Regime Confirmation Panel
When user selects archetype → modal shows 7 regime params (τRMS, N, K, Angular Spread, σ, band, width) with their default ranges for that archetype. User can adjust within bounds. Clicks "Confirm — Apply to Physics Model" before values enter phyR(), snrMod, GI constraint, MU-MIMO gain, optimizer frozen/search split.
**Why**: PDF §02: "the user estimates the bucket" — must be explicit and auditable. User's explicit request.

## Archetype Data Structure Schema
```javascript
const ARCHETYPES = {
  densePublic: {
    label: 'Dense Public Venue', examples: 'stadium · arena',
    dominant: 'Capacity + fairness',
    regime: { tRMS_ns:[50,120], n:[3.5,4.2], K_dB:[-2,2], angSpread_deg:[100,180], sigma_dB:[6,12] },
    apCount: 280, dlFraction: 0.75,
    slaTargets: { posLat:10, vipMbps:25, teaMbps:5, ploss:0.1, airtime:70 },
    slaRanges: { posLat:[3,30], vipMbps:[10,100], teaMbps:[2,50] },
    frozenParams: ['gi','muStreams','clientAgeout','roamCacheAge','mloMode','mloLinks','mloSteer','bsMode'],
    searchParams: ['atFair','txPwr','minRSSI','rxSop','cwBe','maxCli','mcsFloor','bw5g','fragThr','dtim','rtsThr','txopBe'],
    stdPreset: { /* sourced from OPT object line ~1256 */ }
  },
  highDensityEnclosed: { label:'High-Density Enclosed', examples:'lecture hall · office',
    dominant:'Capacity + stability',
    regime:{ tRMS_ns:[80,150], n:[2.8,3.5], K_dB:[3,8], angSpread_deg:[40,120], sigma_dB:[4,8] },
    apCount:150, dlFraction:0.65 /* ... */ },
  sparseHighThroughput: { label:'Sparse High-Throughput', examples:'warehouse · plant',
    dominant:'Coverage + roaming',
    regime:{ tRMS_ns:[150,300], n:[2.5,3.2], K_dB:[5,15], angSpread_deg:[20,60], sigma_dB:[2,6] },
    apCount:40, dlFraction:0.60 /* ... */ },
  mixedRetail: { label:'Mixed Retail / Hospitality', examples:'store · hotel',
    dominant:'POS SLA + guest seg.',
    regime:{ tRMS_ns:[100,200], n:[3.0,3.8], K_dB:[2,6], angSpread_deg:[50,100], sigma_dB:[5,10] },
    apCount:60, dlFraction:0.65 /* ... */ },
  latencyCritical: { label:'Latency-Critical Low-Density', examples:'medical · control',
    dominant:'Jitter / latency',
    regime:{ tRMS_ns:[50,100], n:[2.4,3.0], K_dB:[8,15], angSpread_deg:[10,40], sigma_dB:[2,5] },
    apCount:30, dlFraction:0.70 /* ... */ }
};
let currentArchetype = 'densePublic';
let SLA_THRESHOLDS = { ...ARCHETYPES.densePublic.slaTargets }; // single source of truth
```

## Item 7 — Check Robustness clarification
Two separate optimizer buttons:
- "Apply to Simulator" → sets 12 search knobs to optimizer best values → calls update()
- "Check Robustness" → runs MC on *current* simulator settings (not optimizer output); reports P95 vs SLA thresholds
Iteration count: reads live from #mc-iter dropdown (NOT hardcoded 500). Label shows "Check Robustness (200-draw MC)" updating dynamically.

## Item 3 — Resizable panel
Fix: CSS `resize:vertical; overflow:auto; min-height:80px` on Service Class SLAs container (lines 443-452).
Native browser drag handle on bottom-right corner. No JS needed.

## Item 9 — Reset/Worst Case confirmed
Code audit: "↺ Reset" calls resetAll() = WORST CASE preset. "★ Stadium Optimal" calls applyOptimal() = golden set.
Sprint 2: relabel both honestly. "▼ Load Worst Case" stays in header (useful for Gap Analysis demo).

## Item 18 — SLA drill-down cards
4 cards only: POS, VIP, Teams, Guest. No 5th "PASS" card (confirmed from HTML lines 447-450).

## Sequenced 19-Item Implementation Table (updated June 4)

| # | Pass | Item | Key functions/lines | Size | PDF rationale |
|---|---|---|---|---|---|
| 1 | P1 | Full Config z-index 1000→5000 | CSS ~493 | S | Stacking |
| 2 | P1 | SLA fonts .sn/.ss2/.sd 8/10px→11px | CSS 94–96 | S | Readability |
| 3 | P1 | Below-SLA panel scrollable | CSS ~462 | S | UX |
| 4 | P1 | MC description text (model uncertainty, not measurement) | HTML ~421 | S | PDF §07 Track A |
| 5 | P1 | Pareto α + σ% .kd descriptions | HTML ~410,416 | S | Links to §03 stochastic model |
| 6 | P1 | "Apply Optimized Knobs" relabel + subtitle | HTML ~1657 | S | PDF §07 Track A3 |
| 7 | P1 | "Check Robustness (P95 vs SLA)" relabel | HTML ~1658 | S | PDF §03 chance constraint |
| 8 | P1 | ★ Apply Golden Set + ▼ Load Worst Case relabels | HTML 158,162 | S | Audit provenance |
| 9 | P1 | STABLE badge 3-tier ≥80/60/<60 | JS 867–874 | S | §03 QoE objective |
| 10 | P2 | SLA_THRESHOLDS central object (6 sites) | compute(), slaC(), runMC(), findBreak(), renderGap() | M-L | **Foundation** — §03 archetype-specific SLA |
| 11 | P2 | MC custom percentile (store raw arrays) | runMonteCarlo() + HTML | M | §03 robust form: user sets k-percentile |
| 12 | P2 | Optimizer Target %ile separate input | runOptimizer() + HTML | M | §04 two verbs — MC ≠ optimizer evidence |
| 13 | P2 | DL/UL airtime display (shared channel, display split) | band(), compute() | M | §03 uplink-weighted demand for sub-classes |
| 14 | P3 | ARCHETYPES JS constant (5 physics-grounded) | new constant block | L | PDF §02 Table V0 verbatim |
| 15 | P3 | Archetype selector UI (5 buttons + dominant concern tag) | HTML left panel | M | PDF §02 regime selector |
| 16 | P3 | Regime Confirmation Panel modal (7 params, user adjusts, Confirm btn) | JS setArchetype() + modal HTML | L | **Core**: user acknowledges regime before physics runs. PDF §02 "user estimates the bucket" |
| 17 | P3 | Optimizer reconciliation with archetype frozenParams | runOptimizer(), OPT_SEARCH, OPT_LOCK | M | PDF §01: lock monotone/gate knobs per archetype |
| 18 | P4 | SLA card drill-down (contribution heatmap + Apply fix buttons) | showSLADrillDown() + overlay | L | PDF §05 Tier-0 prior: physical rationale per driver per archetype |

## DL/UL Physics Decision
Single 802.11 channel → shared airtime. One `u = demand/effCap`. DL/UL split is demand-profile only:
`airDL = air × dlFraction`, `airUL = air × (1-dlFraction)`. Display separately. Single `u` drives latency.
NOT two separate capacity pools (would double-count spectrum).

## Open Clarifications (needs user answer before coding)
1. Item 3: which container is "below SLA inputs" — right panel, gap table, or .slabar?
2. Item 18: "PASS" card — Passpoint? Security? Not in current code (4 cards). Confirm 5th card identity.
3. Item 7 robustness: 500 iterations fixed, or matches user's #mc-iter dropdown?
4. Item 16 regime panel: user adjusts τRMS/N/K etc. within bounds → does this also change GI constraint immediately visible in the param sliders, or only affects physics internally?

## Fidelity Ladder Position (from PDF §09)
Sprint 2 stays at Rung 1 (independent multiplicative model) but lays scaffolding for Rung 4 (context modulation by the seven). The regime confirmation panel + ARCHETYPES constant IS Rung 4's foundation. Rungs 2-3 (non-monotonic + interaction terms) are Track B / Sprint 3+.

## Why: Key Strategy Paper Citations
- §00: "unit of the golden-set library is the archetype, not the site"
- §01: "real golden-set decisions are in rows 3-5 [threshold, interaction, context-modulated]"
- §02: "seven are a regime selector — user estimates the bucket"
- §03: proportional-fairness utility + POS SLA chance constraint = the objective the optimizer serves
- §04: optimization ≠ calibration — two verbs, two evidence trails, never mix
- §07 Track A: archetype matrix → MC tab → optimizer tab (this is the Sprint 2 build sequence)
- §09: Sprint 2 = still Rung 1, but scaffolding Rung 4
