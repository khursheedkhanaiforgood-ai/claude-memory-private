---
name: DigitalTwinEngine — PacketCapture Architecture Decisions (June 10 2026)
description: Sub-Sprint 2e complete. Rolling loop design, Wireshark session-end-only, three-layer architecture, disk budget, sprint map through Option 3
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
# PacketCapture Architecture — Locked June 10 2026

## Sub-Sprint 2e — COMPLETE

Files built:
- `live_monitor/agents/capture_agent.py` — CaptureAgent 10-step flow, CaptureResult, FILTER_PRESETS, _open_wireshark
- `live_monitor/storage.py` — pcap_captures table, write_pcap_capture(), list_pcap_captures()
- `live_monitor/dashboard.py` — two-tab layout (Tab 1: Live Metrics, Tab 2: Packet Capture), _capture_tab_layout(), 3 callbacks
- `live_monitor/main.py` — instantiates CaptureAgent, wires _on_capture_complete → SQLite + DataStore

## Three small gaps still pending (do at morning test session start)

- G_A: Add `open -a Wireshark` (no file) at step 4 of _run() in capture_agent.py (~3 lines)
- G_B: Add 256-byte file size guard in _run() after SFTP pull — set error_kind='empty_capture'
- G_C: Add session-end Wireshark loop in main.py finally block (iterate session pcap captures, open fault-tagged ones)

## Wireshark Rules — FINAL (locked June 10)

### Manual single capture (existing):
1. At capture START → `open -a Wireshark` (app opens, welcome screen, no file)
2. At capture END → `wireshark <file.pcap>` (opens the saved .pcap)
3. At session END → Wireshark opens fault-tagged captures from the session (see rolling loop below)

### Rolling loop captures (Sprint 2f, to be built):
- NEVER open Wireshark per-window during the loop — no popup every 60s
- All rolling captures are completely silent during session
- Only at session END does Wireshark open — and ONLY for windows where pcap_inspector found faults
- If 5 windows captured, 2 had faults → 2 Wireshark windows open at session end
- Wireshark toggle = "open fault windows at session end" NOT "open every 60s"

## Session Start Prompt (Sprint 2f — to be built in main.py)

Two questions at startup:
1. "Run continuous packet capture alongside RF polling? [y/n]"
   → Interface: wifi0 / wifi1 / both
   → Capture mode: light (data_only, wifi0) / standard (data_only, wifi0+wifi1) / full (all, all interfaces)
2. "Open Wireshark at session end for fault windows? [y/n]"

If yes to capture: CaptureAgent rolling loop starts immediately alongside CollectorAgent.
If no: Tab 2 manual capture still available on demand.
Wireshark toggle also available as checkbox in Tab 2 during session (can change mid-session).

## Rolling Loop Architecture (Sprint 2f)

- CaptureAgent.start_rolling(interface, filter_preset, on_window_complete) — new method
- Loop: capture 60s → save ring_{interface}_{ts}.pcap → call on_window_complete → restart
- File naming: `ring_wifi0_20260610T143200.pcap` (ts = window start time, ISO slug)
- All files stored in pcap_library.db with session_slug FK
- open_wireshark=False for ALL rolling captures

## Three-Layer Architecture

1. **Collection layer**: Rolling 60s captures (CaptureAgent loop), timestamped, stored in pcap_library.db
2. **Correlation layer**: Post-session (or on-demand), CorrelationAgent runs pcap_inspector on each window + PHY metrics → flags fault windows
3. **Navigation layer**: Session HTML report — PHY timeline + 60s pcap window bands overlaid + fault annotations + one-click Wireshark per fault window

## Session HTML Enhancement (Sprint 2f/3)

Existing: PHY timeline (CRC%, retry%, noise floor, link score over time)
New addition: overlay 60s capture windows as labeled bands on same timeline
Fault annotation: window 2 flagged → colored band + "14 MIC failures, 2 deauths" label
End-of-session: `generate_session_html()` calls pcap_inspector on each session window → annotates HTML

## Bidirectional Correlation

PHY → L2/L3: CRC% spike at T → find ring file covering T → pcap_inspector → MIC failures / retry floods
L2/L3 → PHY: pcap_inspector finds deauths in window N → look at PHY timeline for window N → was CRC% elevated?

## Disk Budget (must implement in Sprint 2f)

Light (data_only, wifi0 only): ~5-8 MB / 60s window → 30-min session = ~150-240 MB
Standard (data_only, wifi0+wifi1): double above
Full (all frames, all interfaces): ~15-30 MB / window → up to 1.8 GB / 30 min

Controls:
- Max disk budget per session (default: 500 MB) — rolling loop stops adding windows when exceeded, warns user
- Capture mode selected at session start prompt

AP load warning: if total_cu_pct > 60% during session, log warning (continuous capture competes with real traffic on loaded AP)

## Sprint Map — FINAL (June 10 2026)

| Sprint | Deliverable | Status |
|--------|-------------|--------|
| 2a | storage.py + report.py + /health | ✅ Done |
| 2e | PacketCaptureAgent + Tab 2 dashboard (manual 60s burst) | ✅ Done |
| 2b | knob_panel.py — dash-daq 4×3 optimizer gauge grid | ← NEXT after morning test |
| 2c | federated_debug.py + synthetic injection | Pending |
| 2f | pcap_library.db + rolling loop + session-start prompt + session-end correlation + HTML timeline overlay | Pending |
| pre-3 | docs/archetype_priors.md — calibration priors from Cisco/Mist | Pending |
| 3 | SimulatorAgent + CalibrationAgent + OptimizerAgent + FederatedTwinAgent + CorrelationAgent (on-demand + PHY-alert) | Pending |
| ICD | icd_reference.html — all 8 ICDs + L2/L3↔PHY appendix | Pending |
| 4 | Prius FederatedDashboard + /api/* endpoints | Pending |
| Mgmt | /config + Bearer token + multi-AP inventory + watchdog | Pending |
| 5 | Railway deploy — pcap_library.db local-only, never ships | Pending |
| Final pcap | Option 3 — remote sniffer streaming, zero-gap ring buffer via SSH pipe | Pending |

## Morning Session Plan (June 10 ~1hr from now, 2hr session)

### Pre-test fixes (15 min)
1. capture_agent.py: add `open -a Wireshark` at step 4 (G_A)
2. capture_agent.py: add 256-byte file size guard (G_B)
3. Confirm `python3 main.py` starts without errors

### Test 1 — Single manual capture end-to-end (25 min)
1. main.py → browser opens at localhost:8050
2. Tab 1: confirm radio metrics polling (CRC%, noise floor, etc.)
3. Tab 2 → select wifi0 + data_only → click Start
4. Verify: Wireshark app opens immediately
5. Verify: countdown shows (⏺ CAPTURING — 57s remaining...)
6. Verify: transitions to ⇩ TRANSFERRING...
7. Verify: returns to ● IDLE
8. Verify: .pcap file in data/ dir with correct timestamp slug
9. Verify: Wireshark opens the .pcap file
10. Verify: file appears in "Captured Files" table

### Test 2 — Validate pcap content in Wireshark (20 min)
- Apply display filter: `wlan` → see 802.11 frames
- Apply: `wlan.fc.type == 2` → data frames only
- Check: timestamps align with capture start time
- Try filter: `wlan.fc.retry == 1` → retransmissions (educational)

### Test 3 — dhcp filter + "🦈 Open" button (10 min)
- New capture with filter: dhcp
- Close Wireshark manually
- Click "🦈 Open" in dashboard table → verify Wireshark opens correct file

### If any test fails → debug before moving to 2b
### If all tests pass → Sprint 2b begins (knob_panel.py)

## Strategic Value (user statement, June 10 2026)

"No one except deep technical RF engineers opens the physical layer in APs the way we are.
If we establish a validation flow between L2/L3 and Physical Layer data — abstracted via
optimization/calibration — that would be a bonus to the industry as a whole."

Three use cases: (1) calibration validation, (2) problem diagnosis, (3) tutorials/learning.
Bidirectional: PHY → L2/L3 AND L2/L3 → PHY.
pcap = diagnostic + educational layer, NOT primary calibration signal.
