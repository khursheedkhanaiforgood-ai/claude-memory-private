---
name: Today's Session Plan — May 7 2026
description: Sequenced workstreams: gap fill (DONE — May 4 EOD pushed b9b4029), DHCP troubleshooting (NEXT — pending user input), then VOSS 4-principle cheatsheet + SW2 config (awaiting EXOS configs from user), then WiFi DT Sprint 1 Day 1.
type: project
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Session Date
**2026-05-07** (Wednesday)

## Sequenced Workstreams (user-directed order)

### ✅ Workstream 1 — Fill memory gap (DONE)
- Created `project_eapol_forensics_may4.md` capturing May 4 EOD content
- May 4 EOD copied to `5320-onboarding-agent/docs/session_summary_20260504.html`
- Committed `b9b4029` on `feature/auto-deploy-agent`, pushed to origin
- ⚠️ Not on gh-pages, index.html not updated (matches Apr 30 EOD pattern in this repo)

### ✅ Workstream 1b — DHCP Troubleshooting (DIAGNOSED + EOD PUBLISHED)
**Live URLs (verified 200 OK May 7 evening after Workstream 1c fix):**
- Today's EOD: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260507.html
- May 4 EAPOL prequel: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260504.html
- Apr 30 VOSS: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260430.html
- Session log MD: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_log_20260507.md
- Pcaps + dialogue: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may7-dhcp/

**Commits:** `2d2bab8` on feature/auto-deploy-agent, `6ec1107`+`b7280ac` on gh-pages, `dcccb27` on main.

### ✅ Workstream 1c — Landing-page fix (DONE May 7 evening)
**Problem found:** May 7 EOD URL returned 404. Root cause: GitHub Pages for `5320-onboarding` serves from the **`main` branch**, but the EOD files (Apr 30 / May 1 / May 4 / May 7) had been pushed only to `gh-pages`. Pages never read `gh-pages`.

**Fix (commit `dcccb27` on main):**
1. Cherry-picked onto main from gh-pages: `session_summary_20260507.html`, `session_summary_20260504.html`, `session_log_20260507.md`, full `data/may7-dhcp/` folder (5 pcaps + 2 tech-support dumps + dialogue docx)
2. Added EOD cards for **Apr 30, May 1, May 4, May 7** to all 3 landing pages (`index.html`, `index-nyt.html`, `index-harpers.html`) — preserves index parity convention
3. Added May 7 session-log card to all 3 pages

**Verified after Pages rebuild:** all 4 EOD URLs + session log + data/may7-dhcp/ pcap = HTTP 200.

**Important fact for future sessions:** `5320-onboarding` Pages source = **`main` branch root** (not gh-pages). If you push EOD HTML to gh-pages it won't go live — must push to main. gh-pages exists as a parallel/backup branch. See `reference_5320_repo_pages_config.md`.
- **Full incident captured** in `project_dhcp_macbook_incident.md`
- May 6 lab failure: MacBook can't DHCP on either SSID, iPhone broke too, reboot fixed it
- Socratic walk complete:
  1. Alex/NotebookLLM "MAC flap" hypothesis → REJECTED (3 different MACs, no flap possible)
  2. 3-layer framing (RF / AP-internal / wire) → AP-internal selected
  3. 4-table elimination (a/b/c/d): bridge FDB floods so (c) fails; TX queue is downstream-only so (d) fails; (b) WPA2 key state wins (bidirectional, silent, per-client, reboot-clearable)
  4. Plot twist (iPhone also broke) extended (b) to GTK rotation wedge
- **Tie-in to May 4 EAPOL EOD:** Concepts #4 (TK install) + #5 (GTK Impact) directly explain the failure mode
- **Ready for EOD HTML drafting** — user requested EOD next

### Workstream 2 — 5320 VOSS+IPE: 4-Principle Cheatsheet + SW2 Config
**User asked for two-part deliverable when we get here:**
1. **Outline EXOS→VOSS cheatsheet (4 principles)**
2. **Walk through each principle to configure SW2** (live, connected to IPE↔EN_RDC) with Corp + Guest VLANs

**Port plan locked (re-confirmed May 7):**
- Port 1/1 → IPE (uplink to RDU DC — already operational)
- Port 1/3 → AP2 (AP3000)
- Port 1/5 → MacBook / corporate user

**State:** SW2 (KhKLab-SW-01) already onboarded via IPE with RDU DC. Phase 1b + Phase 2 ✅ DONE.
**Awaiting:** User said they will share current EXOS configs as input for translation.

**Carry-over from May 1 + May 4 EOD Panel 8 P1/P2:**

P1 (Immediate):
- [ ] Confirm `router ?` syntax in config-dhcp mode on SW2 (KhKLab-SW-01)
- [ ] `show ip interface` + `show ip arp vrf GlobalRouter` — verify VLAN 4047 on Port 1/1
- [ ] Discover **IPE lan1 IP** via IPE Edit Configuration UI (required before default route)
- [ ] Run Service-at-the-Edge CLI block:
  ```
  configure terminal
  router isis
    manual-area 49.b0b1
  exit
  vlan create 70 name "Corp_New" type port-mstprstp 0
  vlan create 80 name "Guest_New" type port-mstprstp 0
  vlan i-sid 70 100070
  vlan i-sid 80 100080
  ```

P2:
- [ ] DHCP config for VLAN 70 (10.70.0.0/24) and VLAN 80 (10.80.0.0/24)
- [ ] IPE transit — **GRE tunnel SW2 → RDU Raleigh** (Q2 resolved May 4: protocol = GRE)
- [ ] VIQ policy — push Corp/Guest SSIDs on VLAN 70/80 to AP2 via FA
- [ ] Fabric Attach on Port 1/3: `auto-sense enable`, `fa enable`
- [ ] Verify: `show fa assignment` (Port 1/3, Active, Dynamic, I-SIDs 100070/100080)

P3:
- [ ] ACL GUEST_ISOLATION verification on BOTH switches

### Workstream 3 — WiFi Digital Twin Sprint 1 Day 1
**Status:** LLD Phase 1 COMPLETE (Apr 29). Sprint 1 coding NOT YET STARTED.

Day 1 Bootstrap:
```bash
cd /private/tmp/wifi-mastery
mkdir -p src/{models,engines,agents,api,rag}
touch src/__init__.py src/models/dpm.py src/engines/link_budget.py src/engines/airtime.py
```

Modules to begin (per LLD):
- [ ] **L1 Data Models** — Pydantic v2 schemas, DPM Sections A–H (use WiFi7Params with `multi_ru_enabled` + `restricted_twt`)
- [ ] **L2 Link Budget Engine** — SNR→MCS, WiFi6LinkBudget (MCS 0-11), WiFi7LinkBudget (MCS 0-13), ComparisonEngine
- [ ] **L3 Airtime Calculator** — Rev Wi-Fi formula + OFDMA ×1.30 modifier + VoIP trap

Branch strategy (from `project_wifi_branch_strategy.md`):
- `feature/wifi6-baseline` (MCS 0-11)
- `feature/wifi7-complete` (MCS 0-13, MLO, 320MHz)
- Mixed-mode deferred to Sprint 4+

## Quality Gates (per global config)
- [ ] `/plan` before any feature > 1 hour
- [ ] TDD on all new WiFi DT modules
- [ ] `/code-review` before push
- [ ] EOD HTML at session end → push to GitHub Pages

## Reference Memory Files
- `project_5320_new_arch_voss_ipe.md` — full SW2 state, IPE details
- `project_eapol_forensics_may4.md` — May 4 EAPOL session, sets next-step list
- `project_wifi_lld_phase1.md` — LLD Phase 1 complete, Sprint 1 ready
- `project_wifi_digital_twin_sprint1.md` — Sprint 1 plan
- `project_wifi_dpm.md` — DPM Sections A–G (~60 params)
- `project_wifi_branch_strategy.md` — wifi6 vs wifi7 branch split
