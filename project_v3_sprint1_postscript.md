---
name: Simulator v3 Sprint 1 — Post-Script for EOD HTML
description: Table of 4 Sprint 1 physics fixes with equations, reasoning, expected outcomes. For EOD HTML post-script section.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---

## Sprint 1 Fixes — June 4 2026
**File:** `docs/stadium_wifi_simulator_v3.html`  
**Branch:** `feature/simulator-v3` (v3 published to main for GitHub Pages review)  
**Commit:** `6997116`

## Fix Table (use verbatim in EOD HTML)

### C1 — MU-MIMO double-count removed

| Field | Detail |
|---|---|
| **Before** | `effPHY = phy × muF` where `muF = muStreamGain[muStreams]` (2.8× for 8 streams); `effCap = effPHY × cm` |
| **After** | `muGain = {1:1.0, 2:1.10, 4:1.15, 8:1.20}`; added to `capMult()` as MAC efficiency term; `effCap = phy × 0.45 × cm` |
| **Reasoning** | `phyR(ss=4, bw, mcs, gi)` already returns the 4-stream aggregate rate. MU-MIMO does not increase PHY capacity — it allows the AP to serve multiple clients in one TXOP (reducing overhead). Multiplying by 2.8 counted spatial streams twice: once in phyR and once in muF. |
| **Expected fix** | 2.8× spurious inflation removed. MU-MIMO still contributes 10–20% efficiency gain in capMult (physically correct). |

### C2 — capMult ceiling + MAC overhead baseline

| Field | Detail |
|---|---|
| **Before** | `capMult()` returned raw product of 14–16 multipliers stacking to **4.62×** at optimal settings. `effCap = effPHY × cm` |
| **After** | `return Math.min(m, 2.0)` added to capMult(); `effCap = phy × 0.45 × cm` |
| **Equations** | `MAC_EFF = 0.45` (DIFS + SIFS + ACK + preamble + backoff ≈ 55% channel overhead at typical 1500-byte frames). `effCap = phyR(4SS, 160MHz, MCS7) × 0.45 × min(cm, 2.0)` = `2882 × 0.45 × 1.38` = **1,789 Mbps** per AP at ATF=strict |
| **Reasoning** | No MAC optimisation set can push effective throughput beyond ~2× the overhead-adjusted baseline. The 4.62× product implied the AP was delivering 4.62× its physical layer capacity — impossible. The 0.45 constant comes from 802.11 frame exchange overhead analysis: DIFS(34µs) + backoff + preamble(20µs) + SIFS(16µs) + ACK(28µs) typically consume 50–60% of channel time for short frames in a dense network. |
| **Expected fix** | `capGbps` drops from ~46,000 Gbps → **1,400–3,500 Gbps** (physics-correct range for 280 AP5060s). Airtime rises from 2% → **20–40%** at kickoff with optimal settings. |

### C3 — Goodput and airtime use same capacity base

| Field | Detail |
|---|---|
| **Before** | `u = demand / effCap` (where effCap = phy × muF × cm); `gp = phy × 0.65 × (1−u) / clients` |
| **After** | `u = demand / effCap` (effCap = phy × 0.45 × cm); `gp = effCap × (1−u) / clients` |
| **Reasoning** | If airtime `u` is defined as `demand/effCap`, the available goodput per client must be `effCap × (1−u) / clients` — the unused fraction of the same capacity pool. The original formula used raw `phy × 0.65` in the gp calculation, which is a different base than effCap. This created an internal contradiction: airtime and throughput reported against different capacity models. |
| **Expected fix** | At u=29%, 75 clients: `gp = 1,789 × 0.71 / 75` = **16.9 Mbps/client** (realistic WiFi 7 stadium goodput). Previously gp was derived from a different base and was internally inconsistent with the airtime value shown. |

### L1 — AP Count slider

| Field | Detail |
|---|---|
| **Before** | `const perAP = Math.round(active / 280)` and `capGbps = bands.reduce(...) × 280` — hardcoded in 5 places. User never set it. |
| **After** | `apCount` slider added (range 50–500, default 280); `perAP = Math.round(active / p.apCount)`; `capGbps = ... × p.apCount`; grid + strandPen updated |
| **Reasoning** | AP count is the single most important deployment variable for capacity. Hardcoding 280 meant all capacity outputs were locked to full Lumen Field regardless of what the user was modelling. It also hid a silent bug: the user had no idea 280 was assumed. |
| **Expected fix** | Sliding from 280 → 100 APs drops capGbps by 64% proportionally (e.g. 2,492 → 890 Gbps at optimal). Enables small-venue and phased-deployment simulation. |

## Key Metrics — Before vs After (at kickoff, 72K fans, 55% concurrency, 280 APs, all optimal)

| Metric | v2 (buggy) | v3 (fixed) | Target |
|---|---|---|---|
| Airtime | ~2% | 20–40% | 20–60% |
| capGbps | ~46,000 Gbps | ~2,400–2,500 Gbps | 1,400–3,500 Gbps |
| Goodput/client | internally inconsistent | ~17–20 Mbps | realistic for WiFi 7 |
| AP Count | hardcoded 280 | user-controlled slider | configurable |
| muF in effCap | 2.8× (wrong) | removed | N/A |
| capMult ceiling | 4.62× (no ceiling) | 2.0× | ≤ 2.0× |
