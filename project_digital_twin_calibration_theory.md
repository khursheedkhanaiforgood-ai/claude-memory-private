---
name: DigitalTwinEngine — Calibration Theory (June 9 2026)
description: Two-source calibration architecture, capMult accuracy assessment, 5 archetypes, Cisco/Mist AI RAG sources
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
# DigitalTwinEngine Calibration Theory — Established June 9, 2026

## Core finding: Two categories of simulator equations

**Physics-grounded (accurate):** PHY1SS rates from 802.11be/ax standard, snrToMcs() thresholds (±1dB), GI_EFF ratios, queuing delay M/D/1 model.

**Empirically-uncertain (calibration targets):** capMult() multipliers — ofdmaDL ×1.35, atFair strict ×1.38, proxyArp ×1.07, MAC efficiency ×0.45.

**Confirmed miscalibration:** MAC efficiency 0.45 is too conservative — Cisco measured 58–68% for WiFi 6 stadium. First confirmed calibration target for Sprint 3.

## Two-source calibration architecture

**Source 1 — Physics priors:** IEEE 802.11 standard + Cisco/Mist published white papers + WLPC corpus. Seeds GP priors per archetype before any AP data. Available now.

**Source 2 — Live AP telemetry:** Real AP3000 SSH polling. Environment-specific calibration for lab. Progressively narrows Source 1 uncertainty. Available Sprint 3+.

## Five archetypes requiring separate calibration

1. **Stadium** — extreme dense, bursty social/POS, CU% <70%, rTWT <3ms POS latency
2. **Enterprise** — 20-40 users/AP, video conf + cloud apps, latency <20ms
3. **Hospital** — IoT-heavy, reliability critical, roaming <50ms, PMF required
4. **Warehouse** — sparse, metal multipath, barcode scanners, long-range
5. **Residential** — low density, mixed generations, power save critical

## RAG sources catalogued

20 public documents in `docs/calibration_rag_sources.html`:
- Cisco CiscoLive 2025 HD Wi-Fi Guide, High Client Density TN, Large Public Networks CX, Connected Stadium
- Mist AI: SLE docs, documentation hub, Juniper analytics
- Wi-Fi Alliance: Wi-Fi 7 Overview (Jan 2024), Wi-Fi 6 Highlights, Data Elements, HaLow
- Extreme Networks: AP5060 stadium blog, NFL deployments, Super Bowl LVII
- MediaTek Wi-Fi 7 + MLO papers, Intel 6GHz tutorial, Anritsu 802.11be
- Super Bowl LVIII 34.8TB record, PwC FIFA WC 2026 spec

**Local WLPC corpus:** T1/T2/T3 at `/Users/khukhan/Downloads/WLPC Troubleshooting/` — private, never commit.

## Pre-Sprint 3 action items

1. Create `docs/archetype_priors.md` — [min, max] for each of 7 uncertain capMult() multipliers per archetype, sourced from Cisco/Mist white papers
2. Update stadium MAC efficiency: 0.45 → [0.55, 0.68] in `compute()` in stadium_wifi_simulator_v3.html
3. First 3 docs to read: Cisco CiscoLive 2025, Wi-Fi CERTIFIED 7 Overview, Extreme AP5060 stadium blog

**Why:** Without archetype priors, GP in Algorithm 5 (BO Pre-Search) starts uninformative. Informative priors from published data give 5–10× convergence speedup.
