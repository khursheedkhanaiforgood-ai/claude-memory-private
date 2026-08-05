---
name: 5320 Onboarding Agent — Full Project State
description: Extreme Networks 5320 lab. SW1=EXOS, SW2=VOSS FE 9.3.2.0. PPSK VLAN split confirmed May 15 (Staff→VLAN1, Students→VLAN10). XIQ→EP1 sprint active. 802.1X/NPS deferred. Last updated May 15 2026.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Switch Details
- **SW1**: 5320-16P-2MXT-2X — EXOS SwitchEngine 33.5.2b118 — 192.168.0.28 — admin/Extreme01!!
- **SW2 (KhKLab-SW-01)**: 5320-16P-2MXT-2X — **VOSS FabricEngine 9.3.2.0** — rwa/<new pw>
  - Serial: /dev/cu.usbserial-A9VKJO11 — baud 115200 — connect via `screen`
  - Console logging: `Ctrl+A H` to toggle screenlog.0
  - VOSS reset: `delete /intflash/config.cfg` + reload (NEVER `unconfigure switch all` — that's EXOS)

## Project Location
- Local: /Users/khukhan/5320-onboarding-agent
- GitHub: https://github.com/khursheedkhanaiforgood-ai/5320-onboarding
- Pages: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/
- **GitHub Pages serves from `main` branch root** (NOT docs/, NOT gh-pages). Push to main to go live.
- Landing pages: `index.html` / `index-nyt.html` (primary) / `index-harpers.html` — keep all three in parity

## Active Branches
- `main` — GitHub Pages source. All EOD HTMLs live here.
- `feature/qos-egress-validation` — May 8 QoS work (parked, ACL fix pending May 18)
- `feature/auto-deploy-agent` — older working branch
- Main worktree: `/private/tmp/main-5320` (used for direct main pushes)

## SW2 VOSS Lab State — May 11 2026 (FishBowl / RDU)

### VOSS 4-Principle Map (KhKLab-SW-01)
| Principle | Key Config | What It Does |
|-----------|-----------|--------------|
| **Brain** (IS-IS) | `ip-source-address 10.159.4.1` (Loopback 1) · `is-type l1` · `redistribute direct` | SW2's fabric identity. Advertises VLAN SVIs into SPB. |
| **Muscle** (SPB) | `spbm ethertype 0x8100` · `nick-name 0.00.02` · `multicast enable` | MAC-in-MAC forwarding. B-VLANs 4051/4052 auto-created. |
| **Service** (I-SID) | VLAN→I-SID bindings below | Labels that carry VLANs through the SPB backbone |
| **Edge** (Auto-Sense/FA) | All 20 ports `auto-sense enable` · FA rules below | LLDP FA TLV detection — NOT MAC-based |

### Service Map (I-SID Bindings)
| VLAN | I-SID | Subnet | Purpose | Notes |
|------|-------|--------|---------|-------|
| 153 | 144153 | 10.153.4.0/24 | AP Management | **UNTAGGED/NATIVE** from AP2. Do not touch in XIQ. |
| 154 | 144154 | 10.154.4.0/24 | Wireless Users (802.1X) | **TAGGED** by AP after RADIUS auth. Configure in XIQ. |
| 155 | 144155 | 10.155.4.0/24 | Cameras | FA camera TLV |
| 156 | 144156 | 10.156.4.0/24 | Wired Data | FA proxy-no-auth / data |
| 4048 | 15999999 | — | Onboarding | Unknown/unauthenticated devices |

**CRITICAL — VLAN 153 vs 154:**
- VLAN 153 = AP management = **untagged (native)**. AP2 sends mgmt frames untagged. SW2 Auto-Sense maps to I-SID 144153. Already working. **Leave alone.**
- VLAN 154 = wireless users = **tagged**. AP2 tags after 802.1X auth (RADIUS Attr 81 = "154"). Configure in XIQ tomorrow.

### Auto-Sense FA Rules
```
auto-sense fa wap-type1 i-sid 144153    ← AP → AP-Management
auto-sense fa camera i-sid 144155       ← camera → Camera
auto-sense fa proxy-no-auth i-sid 144156← non-FA proxy → Data
auto-sense data i-sid 144156            ← data device → Data
auto-sense onboarding i-sid 15999999    ← unknown → Onboarding
```

### DHCP Architecture
- SW2 IS the DHCP server (not relay to external)
- Server bound to management CLIP 10.158.4.1
- Self-relay: each VLAN SVI → CLIP on same switch
- 24h leases (86400s) — matches May 8 GTK mitigation
- domain-name: plmsa.local · DNS: 10.152.0.11

### Key IPs
- Loopback 1 (IS-IS source): 10.159.4.1
- Management CLIP: 10.158.4.1
- Name server: 10.152.0.11
- Syslog: 10.152.0.110
- Domain: RDUFishbowl.local

## SW1 EXOS Lab State — May 14–15 2026

### Active Policy: KB_School_AP_VLAN10_VLAN1
- **SSID**: KB_School_Broadcast (single SSID — PPSK steers to VLAN)
- **Staff passphrase** → VLAN 1 (Default) → DHCP from QF-Modem → 192.168.0.x
- **Students passphrase** → VLAN 10 → DHCP from SW1 → 10.10.0.x
- **Status**: CONFIRMED WORKING May 15. Verified via `show fdb ports 3` + `show iparp`.
- SW1 serial: FJ012544G-00233 · IQAgent heartbeat confirmed ~60s cadence

### VLAN Map (SW1 EXOS)
| VLAN | Name | Subnet | DHCP Server | Port 3 Tagging |
|------|------|--------|-------------|----------------|
| 1 | Default | 192.168.0.x | QF-Modem (router) | Native/Untagged |
| 10 | VLAN10 | 10.10.0.x | SW1 (Supplemental CLI) | Tagged |

### Supplemental CLI (KB_School template — idempotent only)
```
configure vlan VLAN10 dhcp-address-range 10.10.0.10 - 10.10.0.254
configure vlan VLAN10 dhcp-options default-gateway 10.10.0.1
configure vlan VLAN10 dhcp-options dns-server primary 8.8.8.8
configure vlan VLAN10 dhcp-options dns-server secondary 1.1.1.1
configure vlan VLAN10 dhcp-lease-timer 86400
```
**Not in template (run once manually post-factory-reset):**
```
enable ports all
enable ipforwarding vlan Default
save config
```

### Pending — SW1
| Priority | Item | Notes |
|----------|------|-------|
| HIGH | XIQ Switch Template — Port 3 fix | Update template so XIQ is authoritative: Port 3 = Default native (untagged) + VLAN10 tagged |
| MED | VLAN100 sprint resume | Khursheed_SW1_VLAN100 — check XIQ lock cleared, redeploy, verify `show dhcp-server` |
| MED | WPA2/WPA3 Transition Mode | macOS Ventura+ grays out WPA2-only SSIDs — add transition mode to KB_School_Broadcast |
| MED | MacBook auto-connect mystery | Joined without passphrase prompt — likely macOS keychain cached PPSK. Forget network, reconnect manually. |

## 802.1X Deployment — DEFERRED (NPS not ready)

### What Boss Is Doing
- Configuring NPS (RADIUS) server backend
- Creating AD user groups
- Activating `eapol enable` on SW2

### What You Do in XIQ (when NPS is ready)
1. Get NPS IP + shared secret from boss
2. Create 802.1X SSID → security WPA2-Enterprise → user VLAN = **154** (tagged)
3. Add NPS as RADIUS server object
4. Push to AP2
5. **Do NOT touch VLAN 153** — leave AP management native/untagged as-is

### Capture Checklist for Tomorrow
| # | Where | Tool | Signal |
|---|-------|------|--------|
| 1 | OTA | Wireless Diagnostics sniffer | Username plaintext in EAP-Response/Identity → EAP-Success → M1/M2/M3/M4 |
| 2 | en0 | Wireshark | DHCP OFFER from 10.154.4.1, client gets 10.154.4.x |
| 3 | NPS | Event Viewer → Security log | Event 6272 = granted, 6273 = denied |
| 4 | SW2 | `show fdb vlan 154` | Client MAC appears on port 1/3 after auth |
| 4b | SW2 | `show vlan members port 1/3` | VLAN 154 added dynamically alongside 153 |

### 802.1X Key Concepts (PEAP-MSCHAPv2)
- **EAPOL** = EAP Over LAN — wrapper for EAP messages over Wi-Fi
- **PEAP** = Protected EAP — TLS tunnel around the password exchange
- **MSCHAPv2** = password verification inside PEAP (NT-Hash challenge-response, never plaintext)
- **MSK** = Master Session Key — generated by NPS, sent to AP via RADIUS. Never OTA.
- **PMK** = first 256 bits of MSK (Enterprise) vs PBKDF2(passphrase) (Personal)
- **4-Way Handshake** = identical in both modes. Derives PTK (TK + KCK + KEK) + delivers GTK.
- Smoking-gun frame: RADIUS Access-Accept with **Attr 81 = "154"**

## Simulators
| Name | URL / File | Covers |
|------|-----------|--------|
| WPA Simulator | https://claude.ai/public/artifacts/89fa9685-e7ae-49c0-91d7-b2940c5b7694 | 4-Way Handshake (WPA-Personal) |
| 802.11be E2E Simulator | https://claude.ai/public/artifacts/29fe810e-8702-448c-b772-a851c611b952 | EAP + 4-Way + KPI dashboard |
| 802.1X / RADIUS Simulator | `dot1x_simulator.html` (main branch) | Full RADIUS/NPS flow + 802.1X + 4-Way with acronym decoder |

**dot1x_simulator.html features (complete as of May 11):**
- 17-step interactive step-through + ▶ Play/Pause auto-advance
- Speed selector: Slow/Normal/Fast; auto-stops at last step; manual clicks pause auto-play
- Restart bug fixed (history arrays cleared on reset)
- Copyright footer
- All 3 simulators linked from index-nyt.html Simulators section

## EOD HTML Files (all on main, GitHub Pages)
| File | Date | Topic |
|------|------|-------|
| session_summary_20260515.html | May 15 | PPSK VLAN split confirmed + XIQ→EP1 architecture design (6 sessions, 15 ADRs) |
| session_summary_20260514.html | May 14 | KB_School AP VLAN split policy deployed + PPSK User Groups + VLAN steering |
| session_summary_20260511.html | May 11 | EXOS→VOSS 4 principles + 802.1X arc + SW2 config embedded + sprints |
| session_summary_20260508.html | May 8 | DHCP Hypothesis B confirmed (GTK rotation, 3-client fingerprint) |
| session_summary_20260508_qos.html | May 8 PM | QoS egress validation (3 findings) |
| session_summary_20260507.html | May 7 | DHCP incident diagnosis + triage runbook |
| session_summary_20260504.html | May 4 | EAPOL forensics + 306K-frame Wireshark + rogue NAV |
| session_summary_20260501.html | May 1 | IPE live + VIQ onboarded |
| session_summary_20260430.html | Apr 30 | VOSS boot + pre-flight |
| session_summary_20260422.html | Apr 22 | Training sprint PASS (all 6 sessions) |

## Sprint Backlog
| Sprint | Week | Item |
|--------|------|------|
| XIQ→EP1 Engine Sprint A | May 19 (Mon) | Day 1: repo + Railway + DB + FastAPI + LangGraph skeleton |
| XIQ→EP1 Engine Sprint B | After Sprint A | React frontend migration (3 days) |
| XIQ Switch Template Port 3 | May 19 | Make XIQ authoritative for Port 3 VLAN tagging |
| VLAN100 resume | May 19 | Khursheed_SW1_VLAN100 — redeploy + verify |
| QoS ACL fix | May 18 | Branch feature/qos-egress-validation. Fix DSCP marking ACL. |
| AP1_Data download | May 18 | Pull AP3000 pcaps + kdebug wsec logs from May 6–8 before reboot |
| 802.1X/NPS | TBD | Waiting on boss to configure NPS + AD groups + eapol enable on SW2 |
| WiFi Digital Twin Sprint 1 | June 1 | DPM models + Link Budget Engine + Airtime Calculator (17 modules) |

## Key Lessons (running list — updated May 15)

### EXOS / XIQ
- **FDB is truth, ARP is history** — `show fdb ports X` for live VLAN state; ARP ages up to 20 min. See `feedback_fdb_vlan_verification.md`.
- **Dual-MAC ARP observation is normal** — same MAC in two ARP tables = tested both PPSK passphrases on same device. Not a bug.
- **Supplemental CLI = idempotent only** — `enable dhcp` and `enable ipforwarding` hang XIQ at 15% on re-run. Run manually post-factory-reset. See `feedback_supplemental_cli_idempotency.md`.
- **Post-factory-reset checklist**: `enable ports all` → `enable ipforwarding vlan Default` → `save config` — before first XIQ deploy.
- **IQAgent heartbeat** — "proxy device-connector unknown POST /health-check/[serial]" every ~60s = normal keepalive, NOT an error. See `feedback_iqagent_heartbeat.md`.
- **VLAN 1 = XIQ management lifeline** — NEVER add Default/VLAN1 to the Routing section in XIQ.
- **${vlan:NAME}** resolves to VLAN ID, not name. Always hardcode VLAN name in Supplemental CLI. XIQ drops underscores (Guest_100 → Guest100).
- **EXOS factory reset** = `unconfigure switch all` (NOT `delete /intflash/config.cfg` — that is VOSS only).
- **AP reboot after policy changes** = use XIQ-triggered reboot (Monitor→Devices→AP→Utilities→Reboot), NOT power cycle.
- **PPSK must be typed manually** — never paste from email/SMS (invisible chars cause silent Join failure).
- **macOS Ventura+ hides WPA2-only SSIDs** — add WPA2/WPA3 transition mode. iPhone unaffected.
- **`arp -an` on macOS** (not `arp -n` — always needs `-a` flag on macOS).

### VOSS / FabricEngine
- `ip name-server` does NOT feed IQAgent DNS — needs DHCP at boot (ZTP+).
- VLAN interface does NOT support `ip address dhcp`.
- Static route needs TWO commands — create + enable.
- Ports NOT in any VLAN by default (unlike EXOS where all ports start in VLAN 1).
- FA discovery = LLDP TLV, not MAC address.
- IS-IS = Brain (topology), SPB = Muscle (forwarding), I-SID = Service, FA = Edge.
- VOSS reset: `delete /intflash/config.cfg` + reload.
- VLAN 153 = native/untagged AP management — do NOT tag it in XIQ.

### GitHub / Lab Ops
- GitHub Pages serves from `main` root — files in docs/ or gh-pages branch will 404.
- Console logging via `screen`: `Ctrl+A H` toggles screenlog.0.
- All EODs must land on main and be linked from all 3 index pages (index.html, index-nyt.html, index-harpers.html) for parity.
