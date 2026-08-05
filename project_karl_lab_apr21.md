---
name: Karl Lab — Horizon Custom Fabrication (Apr 28 EOD)
description: debrief_apr21.html complete — 15 PPTX slides with CLI config blocks, Pure L2 Update deck (12 slides) added. SW2 pure L2 fix resolved Apr 28.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
Full Primary/Secondary L2/L3 lab built and deployed for Horizon Custom Fabrication on April 21 2026.

**Why:** Test lab for KK's engineer development — proving Primary/Secondary architecture, inter-VLAN routing, QoS, and guest isolation ahead of team training.

**How to apply (Apr 28 2026):** SW2 is misconfigured — debug in progress. Fix default route first, then run verification checklist.

---

## Lab Architecture (UPDATED Apr 28 2026)

- **SW1** (FJ012544G-00233, 192.168.0.28) = L3 Core: all SVIs, all DHCP, ipforwarding on all VLANs
- **SW2** (FJ012544G-00483, 192.168.0.11) = Pure L2 access: no SVIs, no DHCP, no ipforwarding
- **Trunk:** Port 10 ↔ Port 10 (SW1↔SW2) — ONLY inter-switch link as of Apr 28
- **SW2 Port 1:** DISCONNECTED from modem as of Apr 28 (was wrongly cabled to modem before)
- **Loop triangle risk:** ELIMINATED — SW2 Port 1 no longer touches modem, so Default VLAN on Port 10 is now safe if needed for SW2 management
- **AP port:** Port 3, Default VLAN untagged (AP boot/management), VLAN 20+30 tagged
- **Wired client:** Port 5, VLAN 10 untagged
- **Uplink (SW1 only):** Port 1, Default VLAN untagged to Quantum Fiber modem (192.168.0.1)

## SW2 Debug Session — Apr 28 2026 — RESOLVED

**Root cause:** SW2 had a DHCP-learned default route to modem (192.168.0.1) AND Default VLAN was not on Port 10 trunk. Result: AP2 couldn't phone home to XIQ, SW2 had no management path.

**Physical change:** SW2 Port 1 disconnected from modem. Only link = SW2 Port 10 ↔ SW1 Port 10.

**Fix applied (both switches):**
```
# SW2
disable dhcp vlan Default                      ← removed DHCP client + bad default route
configure vlan Default add ports 10 tagged     ← VLAN 1 now crosses trunk (safe — no loop)
save configuration

# SW1
configure vlan Default add ports 10 tagged     ← mirror — VLAN 1 on trunk both sides
save configuration
```

**Why VLAN 1 on Port 10 is now safe:** Loop triangle required SW2 Port 1 → modem → SW1 Port 1. That path is gone (SW2 Port 1 disconnected). No loop possible.

**Result:** XIQ up. Both APs online. SW2 pure L2 confirmed (show iproute = 0 routes, no f flag on VLANs).

SW1 verified state: Default 192.168.0.28/24 · VLAN_10 10.10.0.1/24 · VLAN_20 10.20.0.1/24 · VLAN_30 10.30.0.1/24 — all EUf (Enabled, Up, Forwarding).

## VLANs

| VLAN | Name | Subnet | Gateway | SVI Owner |
|------|------|--------|---------|-----------|
| 10 | Corporate Wired | 10.10.0.0/24 | 10.10.0.1 | SW1 |
| 20 | Corporate Wireless | 10.20.0.0/24 | 10.20.0.1 | SW1 |
| 30 | Guest Wireless | 10.30.0.0/24 | 10.30.0.1 | SW1 |

## Critical Commands (Karl Rule + others)

```
enable ipforwarding vlan Default          ← KARL RULE — MUST have or clients get IPs but no internet
configure vlan Default add ports 3 untagged  ← AP management lifeline
configure vlan Default delete ports 10    ← VLAN 1 loop prevention (both switches)
enable dhcp ports 5,10 vlan VLAN_10      ← port 10 MUST be included for SW2 clients
```

## QoS (Deployed)

- SW1 CLI: VLAN 10/20 → qp6 (high), VLAN 30 → qp1 (low)
- XIQ: Corp_20 User Profile → Scheduling Weight 100, Guest_30 → Weight 10

## Guest Isolation (Deployed via XIQ)

Guest_30 User Profile — 3 firewall rules:
1. DENY Guest_Network (10.30.0.0/24) → Corp_Wired (10.10.0.0/24)
2. DENY Guest_Network → Corp_Wireless (10.20.0.0/24)
3. DROP traffic between stations (Guest-to-Guest L2 isolation)

IP Objects created in XIQ Common Objects: Guest_Network, Corp_Wired, Corp_Wireless

## XIQ Network Policy

- Policy name: **AP_April21_KarlLab**
- Corporate_Wireless SSID → VLAN 20, WPA2/WPA3 PSK (⚠ Key Value field was EMPTY — set tomorrow)
- Guest_Wireless SSID → VLAN 30, OWE + Transition Mode (toggle in SSID config for legacy devices)

## Quantum Fiber Static Routes (Configured)

- 10.10.0.0/24 → 192.168.0.28 (SW1)
- 10.20.0.0/24 → 192.168.0.28 (SW1)
- 10.30.0.0/24 → 192.168.0.28 (SW1)

## OWE Answer (Saved)

OWE Transition Mode = run hidden open SSID alongside OWE SSID. OWE devices → encrypted. Legacy devices → fall back to hidden open. XIQ toggle: "Transition Mode for 2.4GHz and 5GHz" in Guest_Wireless SSID config.

---

## EOD HTML Files (Pushed to GitHub)

All at: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/

- **session_log_20260421.html** — 14-section full lab log, 11 docs with excerpts, complete CLI configs, verification checklist
- **session_summary_20260421.html** — dark EOD dashboard: topology, lessons, 11-doc table with download links, OWE answer, pending items
- **lab_20260329.html** — Apr 21 purple link block + sidebar added (links to both above)
- **debrief_apr21.html** — FULLY COMPLETE as of Apr 28. Reachable from ALL THREE landing pages (index.html, index-nyt.html, index-harpers.html) via amber "PPTX Debrief" card in Training Resources section. Contains:
  - 15 PPTX slides (Karl's Lab) each tagged to Apr 22 Socratic lesson + CLI config block at bottom of every slide (CSS: .config-block, c-comment, c-warn, c-strike spans)
  - 9 Tech Blueprint Doc sections (§1–§9): Architecture, Port Mapping, SW1 CLI, SW2 CLI, XIQ Policy, Security/ACLs, QoS, Alex Vault, Modem Routes
  - **Pure L2 Update — Apr 28** sidebar section (12 slides, L1–L12): extracted from Horizon_Pure_L2_Update.pptx, images in pure_l2/ folder. Covers: loop triangle root cause, architectural fix, updated topology, AP2 phone-home challenge, frame journey, SW1/SW2 config execution (strike workaround / add permanent fix), diagnostic matrix, Lesson 08, next steps.
  - Source files: Technical_Lab_Blueprint_Apr21.docx + Horizon_Hybrid_Lab_Debrief_(2).pptx + Horizon_Pure_L2_Update.pptx

Landing page: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/lab_20260329.html
Reachable from index-nyt.html → "Dual-Switch + 4×AP Lab — Mar 29" card → scroll to Apr 21 block

## 11 Documents — All Uploaded to GitHub Repo

| # | File in Repo | Type | Notes |
|---|-------------|------|-------|
| 1 | (file:// link only) | Claude AI | InterVLAN morning session |
| 2 | (file:// link only) | NotebookLLM | InterVLAN synthesis |
| 3 | network_requirements.docx | Customer Brief | TODAY'S STARTING DOC — Horizon spec |
| 4 | Claude_NewLab_Karl_Dialogue_Apr21_v3.docx | Claude AI | Full Socratic build |
| 5 | (file:// link only) | NotebookLLM | KarlLab v1 synthesis |
| 6 | NoteBookLLM_KarlLab_Apr21_v2.docx | NotebookLLM | VLAN 1 loop, LLDP, trunk |
| 7 | Claude_FullDay_Apr21_Until_QoS_Classification.docx | Claude AI | Full day to QoS |
| 8 | NoteBookLLM_FinalItems_Apr21_QoS.docx | NotebookLLM | QoS + isolation strategy |
| 9 | (file:// link only) | Reference | WiFi Overview JJ+KK comments |
| 10 | Technical_Lab_Blueprint_Apr21.docx | KK Synthesis | The Alex Vault — best single reference |
| 11 | ClaudeAI_QoS_Apr21.docx | Claude AI | Full day + XIQ nav + final confirmation |

---

## Pending — Next Session

**FIRST: Run verification checklist** (17 tests in session_log_20260421.html#verification)

| # | Test | Priority |
|---|------|----------|
| CLI-1–6 | VLAN, ipforwarding, Karl Rule, DHCP leases, LLDP, VLAN 1 off port 10 | First |
| TEST-1–4 | DHCP on Corp Wired/Wireless, Guest Wireless | Second |
| ISO-1–4 | Guest→Corp blocked, Guest→Guest blocked, Corp has internet | Third |
| XIQ-1–3 | Both APs online, firewall rules active, QoS weights confirmed | Fourth |

**⚠ Corporate SSID PSK password not set** — Key Value field was empty in XIQ. Navigate: XIQ → Network Policy → AP_April21_KarlLab → Corporate_Wireless → Security → Key Value

**⚠ OWE Transition Mode not yet toggled** — Guest_Wireless SSID → "Transition Mode for 2.4GHz and 5GHz" toggle → push policy

**THEN: Training Sessions 2–6** — ALL DUE APR 22. Team review APR 23.
- Session 2 opening already posed: inter-VLAN ping bridge question
- Sessions 3–6: pending
- Update S4+S5 tracker questions from WiFi Overview doc (DFS + WPA3) before running those sessions

---

## Key Lessons Locked In Memory

1. **Karl Rule** — `enable ipforwarding vlan Default` on SW1. Without it: clients get IPs, no internet.
2. **DHCP reach-back** — `enable dhcp ports 5,10` — port 10 MUST be included for SW2 clients.
3. **VLAN 1 triangle loop** — `configure vlan Default delete ports 10` on both switches.
4. **AP lifeline** — `configure vlan Default add ports 3 untagged` — AP boots dumb, needs untagged VLAN 1 to reach modem and call home to XIQ.
5. **EXOS ACL syntax** — `create access-list "ip-source-address..."` fails. Use `edit policy NAME` (vi editor, `source-address` keyword, Esc+:wq). For this lab: bypassed by XIQ.
6. **Guest-to-guest isolation** — ACLs can't stop L2 intra-VLAN traffic. Must use XIQ "Drop traffic between stations" firewall action on AP.
7. **OWE Transition Mode** — Enable the toggle in XIQ Guest SSID for legacy device backward compat.
8. **XIQ IP Objects** — Must create in Common Objects BEFORE adding firewall rules, or "IP object must be saved first" error.
