---
name: WiFi 7 Digital Twin Sprint Plan
description: E2E pipeline sprint — 1 AP + 1 MacBook, Omniverse → Sionna RT → ns3-Sionna → Streamlit QoE. Phase 0 starts tomorrow (June 17).
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
# WiFi 7 Digital Twin Sprint Plan

**Repo to create:** `/Users/khukhan/wifi-7-twin/`

**Goal:** ONE complete E2E run: 1 AP (AP3000 geometry) + 1 static MacBook client, MacBook CPU via Docker. `./run_twin.sh` → 60s sim → dashboard at `localhost:8501` → PCAP in Wireshark.

**Pipeline:** Omniverse USD Composer (offline scene edit) → export Mitsuba XML → Sionna RT (CIR, CPU/LLVM) → ZeroMQ IPC → ns-3 802.11be MAC → FlowMonitor → `live_telemetry.json` → Streamlit QoE

**Key constraint:** No multi-AP, no mobility, no RRM, no MLO — until Phases 0-3 are green. "First one complete E2E that runs."

## Correct Docker Approach (from repo source)
- Base: `ubuntu:24.04` (NOT tensorflow GPU image) — repo ships `Dockerfile.cpu`
- Multi-arch: runs native ARM64 on Apple Silicon, no Rosetta needed
- ns-3 version: **3.40** (specific tarball from nsnam.org)
- ns3sionna cloned into `contrib/sionna/` inside ns-3 tree
- Python venv at `/opt/sionna-venv` with `requirements_cpuonly.txt`
- Build: `docker build -f Dockerfile.cpu -t ns3sionna-cpu .`

## WiFi Standard — CONFIRMED
- Repo's highest tested standard: **`WIFI_STANDARD_80211ax`** (Wi-Fi 6 / 802.11ax)
- AP3000 hardware = 802.11ax → **perfect match for calibration**
- 802.11be: untested in repo. `SionnaPhasedArraySpectrumPropagationLossModel` = TODO (not implemented)
- MIMO: not available. All current models = SISO

## Three CPU Phases
- **Phase 1 (Static):** 1 AP + 1 MacBook, `simple_room.xml`, 802.11ax, GUI + QoE baseline
- **Phase 2 (Multi):** 2-3 APs, 5-10 clients, VoIP + video + data concurrent
- **Phase 3 (Mobility):** SionnaMobilityModel random walk, roaming QoE
- ALL THREE on CPU before any GPU work

## Phase Status

| Phase | Description | Status |
|-------|-------------|--------|
| 0 | `docker build -f Dockerfile.cpu -t ns3sionna-cpu .` | NOT STARTED |
| 0.5 | Sionna server smoke test — ZMQ port 5555 binds | NOT STARTED |
| 1 | 1 AP + 1 STA + simple_room.xml + 802.11ax + GUI + PCAP | NOT STARTED |
| 2 | Multi-AP + multi-client + multi-service | NOT STARTED |
| 3 | SionnaMobilityModel random walk | NOT STARTED |
| Calibration | Compare twin QoE vs AP3000 15 parameters | NOT STARTED |
| GPU | `docker build -f Dockerfile` + `--gpus all` | DEFERRED |

## Parked for Later
- ns3-Sionna full I/O specification (ZMQ message schema, all tunable params)
- Omniverse I/O specification (what it exports, what Sionna imports)
- Real-room Omniverse scene (after Phase 3 validates pipeline)

## Phase 0 Gate Commands
```
docker run --rm --platform=linux/amd64 wifi7-twin-prod /workspace/ns3sionna/ns3 --version
docker run --rm wifi7-twin-prod python3 -c "import sionna.rt; print('ok')"
```

## Top Risks
- R1: ns3sionna `./ns3 configure` may fail on TF 2.16.1/pybind11 ABI mismatch → pin TF 2.15 as fallback
- R3: `SionnaSpectrumPropagationLossModel` may not exist in upstream → Phase 0.5 grep first
- R5: Mitsuba XML must be version 3.x (not 2.0) — Sionna RT uses Mitsuba 3
- R6: Build time ~40 min under Rosetta (not 20 min as initially estimated)

## Source Documents
- ns3-Sionna paper: arXiv:2412.20524v1 (TU Berlin, Dec 2024, `github.com/tkn-tub/ns3sionna`)
- NVIDIA Gemini dialogue: `/Users/khukhan/Downloads/NVIDIA AI Programming - WiFI DigitalTwin_v1_June 16 2026_GoogleGemini.pdf` (95 pages, private)

## EOD
`docs/session_summary_20260616_digital_twin_sprint.html` — commit c6aeff5 — live at 5320-onboarding GitHub Pages

**Why:** Industry gap — no single tool unifies deep RF physics (Sionna RT) + L2/L3 MAC sim (ns-3). 1 AP + 1 static client is feasible on MacBook CPU because it's only 1 link trajectory, computed once, cached.

**How to apply:** Next session say "start Phase 0 wifi twin" — create repo, write Dockerfile, `docker build`. Do NOT write main.cpp until Phase 0.5 confirms the actual C++ class name.
