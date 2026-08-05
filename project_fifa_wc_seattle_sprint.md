---
name: FIFA WC Seattle WiFi Design — Sprint State
description: EP1 Security dynamic VLAN lab. Sprint 1+3 COMPLETE. Simulator v2 Phase 1 LIVE (60 params, Broadcom AP5060, all EP1 Optional Settings). Stochastic Phase 2 queued for June 3.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
## ⬇ NEXT SESSION — START HERE (June 3)

### Simulator v2 Phase 2 — Stochastic Model (START HERE)
**User's architecture vision (save this — it is precise):**
1. **Monte Carlo** — preferred approach (has prior exposure). Run N iterations with Poisson client arrivals + Pareto per-client demand. Show P5/P50/P95 confidence bands on every metric.
2. **NOC Portal integration** — forward-looking estimates feed a real-time oversight dashboard. Predictions displayed before/during event.
3. **Post-event calibration loop** — align actual (measured) vs simulated → tighten distribution parameters → model improves after each game.
4. **Digital Twin (long-term)** — not simulation. Real-time mirror of the live network, fed by telemetry. Separate from stochastic sim.

**Do NOT discount the other two levels:**
- Quick variance: demand ±N% slider → P5/P50/P95 bands (may be a useful warm-up)
- Time-series animation: match timeline with clients arriving/leaving tick by tick (needed for NOC portal)

**Build order agreed:** Config Export (done) → Monte Carlo (June 3) → NOC portal (future) → calibration loop (future) → Digital Twin (future, separate project).
**Key question for June 3:** Monte Carlo as new tab in v2 or separate file? Recommend: same file, new "Monte Carlo" tab in center panel.

---

### Simulator v2 — Current State (end of June 2 session)

**Parameter count: 60** (was 39 announced → 47 actual → 60 after adding all EP1 Optional Settings)

**Title:** "WiFi 7 Simulator v2 — 60 Parameters, One Breaking Point"
**File:** `docs/stadium_wifi_simulator_v2.html`
**EOD:** `docs/session_summary_20260602_fifa_wc.html`
**Latest commit:** `1e6b355`

**Chip attribution:** AP5060 uses **Broadcom** (NOT Qualcomm/IPQ9574). All labels corrected in simulator + EOD + landing pages.

**The Breaking Point:** **Airtime Fairness OFF** at kickoff (72K fans, 55% concurrency). One MCS0 device at −78 dBm consumes 32× airtime of MCS11 client. Airtime crosses 70% SLA, POS misses 10ms target, score collapses. Largest single-parameter multiplier swing in capMult chain (×1.38).
Second: **Ignore Broadcast Probe OFF** — 50K+ probe requests at kickoff push airtime past 70% before any data frames.

**Full 60 parameter list by section:**
- Demand (5): AP Type, Fan Count, Devices/Fan, WiFi Attach, Concurrency
- PHY (9): TX Power, 5G BW, 6G BW, Guard Interval, MBR, RX-SOP, MCS Floor, Noise Floor, Preamble Puncturing
- MAC/EDCA (22): OFDMA DL, OFDMA UL, MU-MIMO, MU-MIMO Streams ★, BSS Color, rTWT, Multi-RU, MLO, MLO Mode, MLO Links ★, MLO Steering ★, 4K-QAM ★, 802.11r, 802.11k/v, DCS Lock, TXOP VO, TXOP BE ★, CWmin BE ★, RTS/CTS ★, **U-APSD ◆, WMM AC VO ◆, WMM AC VI ◆**
- Admission (8): Max Clients, Min RSSI, Airtime Fairness, Band Steering, **Client Ageout ◆, Fragment Threshold ◆, Roaming Cache Update ◆, Roaming Cache Ageout ◆**
- SSID/Network (12): BC Filter, MC→UC, Proxy ARP, Client Iso, Enhanced RNR, PMF, AirDefense, DTIM, **MC→UC Util Threshold ◆, MC→UC Membership ◆, Multicast Drop ◆, Ignore BC Probe ◆**
- Degradation/Security (4): Degradation Level, Cyber Attack, **MAC DoS Prev ◆, IP DoS Prev ◆**

★ = AP5060 Broadcom chip params (7 total)
◆ = EP1 SSID Optional Settings page (13 total, added end of June 2)

**Spec vs Simulator conflicts (still pending June 3 cleanup):**
| Parameter | Spec Target | Simulator Optimal | Fix |
|-----------|------------|-------------------|-----|
| Guard Interval | 800ns (high mobility) | 1600ns | Change applyOptimal() to 800ns |
| Min RSSI | −67 dBm | −70 dBm | Change applyOptimal() to −67 |
| MLO Traffic Steering | QoS-based | AI-driven | Discuss — EP1 RRM vs spec |
| MCS Floor | MCS 5–7 | MCS 4 | Change applyOptimal() to MCS 6 |

**Params in spec but still missing (add in cleanup):**
1. CWmin AC_VO (3 slots) — only CWmin AC_BE exists
2. Beacon Interval (100 TU) — not exposed
3. SAE PWE Method (H2E) — not in simulator

---

### June 2 — COMPLETE
1. **Simulator v2 Phase 1** — 60 tunable parameters LIVE
2. **Chip attribution fixed** — IPQ9574 (Qualcomm) corrected to AP5060 (Broadcom) everywhere
3. **All EP1 SSID Optional Settings** — 13 new params added (◆ above)
4. **Config Export** — "All 60 Params" gold button, 7 sections including new SECURITY & DoS
5. **EOD HTML** — `docs/session_summary_20260602_fifa_wc.html` LIVE with §10 full 60-param table
6. **Commits:** e0702c4 → 360d43e → cde387c → a3ba4d7 → e86ca71 → b3f1895 → 1e6b355

### June 3 — COMPLETE
1. **Simulator UI overhaul** — title renamed, How-to-Use guide, P50/P75/P90 selector above tabs, center gap fixed, SLA bar readable, log strip 90px
2. **Download Config button** — green ⬇ Download in header, data: URI, self-contained text gen
3. **Full physics audit** — 17 bugs found (3 Critical, 7 High, 4 Medium, 3 Low)
4. **Bug Report addendum** — `docs/simulator_v2_addendum.html` LIVE: bug tracker + verbatim dialogue + sprint plan
5. **EOD commits:** 7eccb13, 4672c9a, 34c5142, 18d2a2d, 9919c4f, + addendum commit

### ⚠️ JUNE 4 — START HERE: Simulator physics is broken
**Say: "start Sprint 1 simulator bugs"** → open `docs/simulator_v2_addendum.html` first
Full sprint plan in: `memory/project_simulator_v2_bugs.md`

**Sprint 1 morning — 5 min prerequisite:** Review QoE vs Scenario curve targets BEFORE coding.
- Target after Sprint 1: airtime 20–60%, capGbps 1,400–3,500 at kickoff with optimal settings
- POS SLA curve should stay above 10ms threshold up to ~40 clients/AP then degrade gracefully
- This is the acceptance criterion — if it still shows 2% airtime after fixes, something is still wrong

Sprint 1 (morning): C1 remove muF from effCap · C2 cap capMult at 2.0× · C3 unify gp/u capacity base · L1 add AP count slider
Sprint 2 (late morning): H2 wire OFDMA UL · H3 normalize splits · H5 fix snrMod.g6 · font audit
Sprint 3 (afternoon): H7 queueing delay · M1 MC label · M2 TX power · M4 rTWT formula

### Pending sprint queue (June 4+):
- **Simulator v2 Sprint 1** — Fix 17 physics bugs (see simulator_v2_bugs.md) ← START HERE
- **Validate all 60 params** — after physics fix, validate all 15 search + 45 locked params against IEEE 802.11be spec and AP5060 behavior; 3 params to promote to search: Guard Interval, MCS Floor, DTIM
- **Agentic framework** — LangGraph orchestrator + 7 sub-agents (Physics, Param, MC, Optimizer, Scenario, Calibration, UI, Report); build AFTER physics is verified. Design: `memory/project_simulator_agentic_framework.md`
- **Simulator v2 Phase 2** — Monte Carlo stochastic model (AFTER physics fix + agentic framework)
- **Track B calibration** — ns-3 + NVIDIA Sionna RT; 4-level validation sequence; surrogate calibration loop. Design: `memory/project_track_b_calibration.md`
- **Spec cleanup** — Fix 4 value conflicts in applyOptimal(); add 3 missing params; promote GI + MCS Floor + DTIM to search space (12 → 15 knobs)
- **Guest CWP verification** — OWE assoc → CWP redirect → self-reg → email → QR → PPSK → VLAN 30 FDB
- **PPSK self-reg email+QR** — configure CWP email/notification tab in EP1
- **PCI-DSS VLAN 50** — ACL lockdown, no inter-VLAN routing, ping isolation proof
- **AD Integration Sprint 2** — BLOCKED waiting for boss AD access
- **Sprint 5** — Fix OpenAI 401 → RAG ingest PPSK + FIFA docs

---

## Simulator v2 — Branch Strategy

### v1 — FROZEN FOREVER
- File: `docs/stadium_wifi_simulator.html` · Branch: `main` · DO NOT MODIFY

### v2 — LIVE on main
- File: `docs/stadium_wifi_simulator_v2.html` · Merged to main

---

## Lab Sprint 1 — COMPLETE (2026-06-01)
- `74:a6:cd:8b:19:60 TeamUSA(0010) port 3` — VLAN 10 ✓
- `74:a6:cd:8b:19:60 VIP(0040) port 3` — VLAN 40 ✓

## Lab Sprint 3 — COMPLETE (2026-06-01/02)
- `b6:1c:de:e9:1e:92 TeamUSA(0010) port 3` — VLAN 10, iPhone, PPSK ✓
- `b6:1c:de:e9:1e:92 TeamAUS(0020) port 3` — VLAN 20, iPhone, PPSK ✓
- SSID: `WC_Seattle-PPSK`, WPA2-PSK + PPSK, Classification: SERVICE

---

## Critical Lessons — Burn These In

### EP1 Security returns Filter-Id (Attr 11) — NOT Tunnel attributes
Assignment Rule Value = policy name string (e.g. `Policy_TeamUSA`), NOT the VLAN number.

### RadSec Proxy in EP1 Security kills all cloud auth
Delete it unless intentionally using local survivability.

### macOS Ventura+ / iOS 17+ — EAP-TTLS/PAP requires mobileconfig
Profile: `docs/WC_Secure_TeamUSA_20260601_1333.mobileconfig`

### Policy_Guest catch-all must be LAST (position 5)

### PPSK Classification must match between User Group and SSID (SERVICE)

### CWP profile type must match SSID auth type (Enhanced Open needs Enhanced Open CWP)

---

## Confirmed Working Configuration

### SW1 VLANs + DHCP
- VLAN 10 TeamUSA: 10.10.0.1/24 · VLAN 20 TeamAUS: 10.20.0.1/24
- VLAN 30 Guest: 10.30.0.1/16 · VLAN 40 VIP: 10.40.0.1/24 · VLAN 50 POS: 10.50.0.1/24

### Test credentials
khursheedkhan@gmail.com / WCSeattle2026!! → Group_TeamUSA → VLAN 10
MacBook MAC: 74:a6:cd:8b:19:60 · iPhone MAC: b6:1c:de:e9:1e:92

### Final SSID Architecture
- **WC_Secure** — WPA3-Enterprise, dynamic VLAN 10/20/40/50 (802.1X)
- **WC_Seattle-PPSK** — WPA2+PPSK, dynamic VLAN 10/20/30, per-user passphrase
- **WC_Guest_CWP** — Enhanced Open (OWE), VLAN 30, self-registration CWP (in progress)
