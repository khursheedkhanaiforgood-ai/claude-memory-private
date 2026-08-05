---
name: Simulator Track B — ns-3 Calibration Architecture
description: How Track A (JS multiplier chain) and Track B (ns-3) relate; calibration loop; 4-level validation sequence; all-60-param ns-3 validation plan
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---

## Intent (June 3 2026)
After Sprint 1 fixes Track A physics bugs, validate ALL 60 parameters with ns-3 — not just the 15 search knobs. Then build the Monte Carlo + optimizer on top of the validated model. ns-3 is the calibration target, not a replacement.

**Why:** User wants confidence that the 60-parameter model reflects real AP5060/WiFi 7 behavior before the optimizer produces "golden sets" for FIFA WC Seattle.

## Track A vs Track B — Key Distinction

| | Track A (JS simulator) | Track B (ns-3) |
|---|---|---|
| Model type | Closed-form expected-value equations | Discrete-event state machines |
| Speed | Milliseconds | Minutes to hours |
| What it captures | Mean of MAC behavior | Full distribution incl. tails |
| EDCA backoff | Simplified formula | Actual random draw from [0,CWmin] |
| Collision events | Implicit in efficiency factor | Explicit — per-frame retry + CW doubling |
| Probe storms | Probe Penalty multiplier | Actual probe frame contention |
| Roaming | Not modeled | 802.11r state machine |
| Use | Optimization sweeps, MC confidence bands | Ground truth, coefficient calibration |

## Why ns-3 Gives Higher Fidelity

802.11 MAC is fundamentally stochastic:
- EDCA backoff draws from [0, CWmin] uniformly → actual collision probability is a random variable
- Retry count + exponential CW doubling → latency is heavy-tailed under congestion (not Gaussian)
- Association/reassociation state transitions → roaming latency spikes are discrete, not smoothed
- Probe request storms → bursty channel occupancy Track A captures only as a multiplier

Track A captures the *mean*. ns-3 captures the *distribution*, including the tail events that actually break SLAs.

## Would Track A Monte Carlo Bands Contain ns-3 Output?

- **P50**: Yes, if Track A physics correct after Sprint 1. Target: within 10-15% after calibration.
- **P95 tail**: ns-3 WORSE — collision cascades, TXOP starvation, probe storms create sharper failure modes than formula predicts. Gap grows with density.
- **P5 (best case)**: ns-3 slightly BETTER — real MAC recovery is more efficient than queueing model.
- **Density sensitivity**: At 1 AP + 5 clients → ~5% gap. At 280 APs + 39,600 clients at kickoff → P95 gap could be 30-50%.

## Calibration Loop

1. Fix Track A (Sprint 1) → physically correct multipliers
2. Run ns-3 with same parameter set θ → get ns-3 QoE distribution (many runs)
3. Compare: Track A P50 vs ns-3 median → calibration error δ per scenario
4. Adjust capMult coefficients to minimize δ across validation scenario set
5. After calibration: Track A P50 should predict ns-3 median within 10-15%

This is **surrogate model calibration**. Track A becomes a fast proxy for the slower, higher-fidelity ns-3.

## 4-Level Validation Sequence (1 AP → Multi-AP → Multi-Client → Mobility)

Each level isolates exactly one new interaction class:

**Level 1: 1 AP, 1 client** — PHY layer baseline
- Validates: MCS selection curve, RSSI→SNR→MCS, TX power effect
- Both models should agree within 5%
- Confirms: Track A PHY rate equations are correct after C1/C2 fix

**Level 2: 1 AP, N clients** — MAC contention
- Validates: EDCA backoff, CWmin effect, Airtime Fairness toggle
- BSS Color has no effect here (single BSS) — pure CSMA/CA behavior
- Key check: does ATF 1.38× coefficient match ns-3 airtime distribution ratio?

**Level 3: M APs, N clients** — spatial reuse + interference
- Validates: BSS Color, OFDMA multi-user gains, inter-AP CCI
- Probe storm magnitude becomes measurable
- MU-MIMO spatial stream gains validate (was C1 bug)

**Level 4: M APs, N clients, mobility** — roaming + transient behavior
- Validates: 802.11r fast transition timing, band steering, client ageout
- Kickoff spike (Poisson arrival burst) stress test
- QoE variance explodes here — roaming events cause 50-200ms gaps
- This is the final validation before "golden set" claims

## 60-Parameter Classification for ns-3 Validation

**Category 1: Formula-validatable** (ns-3 confirms coefficient magnitude, 20+ params)
- OFDMA DL/UL, BSS Color, MBR, MLO, MultiRU, TXOP, RTS/CTS, GI, MCS Floor, TX Power
- Method: run ns-3 toggle ON/OFF per param, measure QoE delta → compare to our multiplier value

**Category 2: State-machine-only** (only ns-3 can validate, ~15 params)
- 802.11r fast BSS transition timing, CWmin AC_BE/AC_VO, U-APSD, WMM priorities, client ageout, roaming cache
- These have no closed-form equivalent — ns-3 is ground truth

**Category 3: Locked/administrative** (validate by inspection, ~25 params)
- PMF, AirDefense, client isolation, MAC/IP DoS prevention, DTIM (mostly)
- Policy controls, not physics — confirm wiring is correct, not coefficient magnitude

## Search Space: 12 → 15 Knobs

Promote these 3 from locked to search (Google AI + analysis both flag these):
1. **Guard Interval** (800ns for high-mobility stadium — currently 1600ns in applyOptimal)
2. **MCS Floor** (significant admission vs throughput tradeoff — currently MCS4)
3. **DTIM interval** (affects multicast + power save overlap at stadium scale)

Do in Sprint 3 spec cleanup.

## ns-3 Code Warning

Google AI generated C++ ns-3 code with fabricated APIs:
- `wifi7.SetMultiLinkOperation(WIFI_MLO_STR)` — does NOT exist in ns-3
- `mobility.SetVelocityAndAccelerationAllocator(...)` — does NOT exist

Real MLO config requires per-link PHY helper with ChannelSettings per link (weeks of work, not one line). When building Track B ns-3 skeleton, write from scratch — do NOT use Google AI's generated code as starter. Verify every API call against ns-3 3.40+ docs before compiling.

**How to apply:** On every Track B discussion: Track A = fast surrogate, ns-3 = calibration target. Fix physics first, validate level by level, calibrate coefficients, THEN claim golden sets.
