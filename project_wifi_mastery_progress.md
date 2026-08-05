---
name: 802.11 Mastery Curriculum Progress
description: Socratic Curriculum Agent state — current phase/day, session log, handoff notes
type: project
originSessionId: 52c71221-9cc9-44db-9c54-689b1e8f03fe
---
## ⬇ NEXT SESSION — START HERE

**Last worked:** 2026-05-28  
**Pick up with:** Phase 1, Day 1 — OFDM subcarrier spacing (Socratic session not yet run)

**Opening question loaded:**
> "802.11ac uses 312.5 kHz subcarrier spacing. 802.11ax dropped it to 78.125 kHz — exactly ¼. Symbol time grew from 3.2 μs to 12.8 μs. Why does narrower subcarrier spacing force a longer symbol time — and what problem does 12.8 μs solve that 3.2 μs couldn't, in a dense indoor multipath environment?"

**Also pending from last session:**
- Unified EOD HTML template (merge May 24 sticky TOC/timeline + May 25 write boundary/protocol flows)
- Fix May 24 EOD URL bug: `khursheedkhanaiforgood` → `khursheedkhanaiforgood-ai`
- Multi-LLM research framework — plan approved, code not started (`/Users/khukhan/multi-llm-research/`)

**EOD HTML for last session:** `https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260528.html`

---

## Current State

**Phase:** 1  
**Day:** 1  
**Status:** NOT STARTED  
**Repo:** https://github.com/khursheedkhanaiforgood-ai/wifi-80211-mastery  
**Pages:** https://khursheedkhanaiforgood-ai.github.io/wifi-80211-mastery/

---

## Curriculum Map (34 days total)

| Phase | Topic | Days | Status |
|-------|-------|------|--------|
| 1 | PHY & RF Fundamentals | 8 | Not started |
| 2 | MAC & Frame Mechanics | 6 | Not started |
| 3 | Security Architecture | 6 | Not started |
| 4 | QoS, Roaming & Convergence | 5 | Not started |
| 5 | Capacity Planning & HDD | 5 | Not started |
| 6 | Field Troubleshooting Capstone | 4 | Not started |

---

## Handoff Notes (Architect → Curriculum Agent)

**Lab gear:** AP3000 (AP1) on IQ Engine OS; EXOS SW1 (SwitchEngine 33.x); VOSS SW2 (FabricEngine 9.3.x); XIQ/EP1 cloud; Wireshark on MacBook; iPhone as client device.

**WLPC corpus tiers (NEVER push to GitHub):**
- T1 authoritative: 802.11ax-2021.pdf, 802.11be-2024.pdf, 80211-2024.pdf, WLANPros v5, WLPC Express 2026 PDF, Revolution Wi-Fi Capacity Planner XLSM, Wi-Fi SSID Overhead Calculator.xlsx
- T2 reference: Aruba VHD Planning, Apple Wireless enterprise docs
- T3 tools: WLAN Pi, Wireshark profiles, PCAPs, dfilters

**Key incidents already diagnosed (background knowledge assumed):**
- May 6: WPA2 TK desync + GTK rotation cascade (MacBook + iPhone, Guest VLAN10)
- May 7: DHCP NAK root-caused via before/after pcap on same Guest LAA MAC
- May 8: DHCP repro — trigger = en8/Port 5 dual-interface event; self-recovery 2-4 min

**Prior knowledge depth:** Khursheed has hands-on AP3000/EXOS/VOSS/XIQ/EP1 experience, has diagnosed EAPOL/DHCP/GTK incidents at packet level, and understands 4-way handshake and PTK derivation. Phase 1 starts from OFDM fundamentals — assume strong practitioner context, not beginner.

**Socratic method:** Ask one or two questions. Wait for answers. Probe gaps. Do not lecture unprompted. Use the lab exercise at the end of the exchange, not the start.

---

## Session Log

*(empty — first session not yet run)*

---

## Pending Deliverables

- docs/study_capstone_report.html — Phase 6 Day 4 output, not yet created (intentional)
- EOD HTML session_summary_20260525.html — DONE (commit 52df80e, all 3 index pages updated)
- Unified EOD HTML template — May 24 (sticky TOC + timeline dots) vs May 25 (protocol flows + write boundary). Merge styles. Deferred 2026-05-28.
