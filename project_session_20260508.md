---
name: Today's Session Plan — May 8 2026 (Friday)
description: DHCP issue repro day. Plan: capture baseline known-good state → trigger TK/GTK corruption deterministically → compare. Socratic question on TRIGGER (A/B/C/D hypotheses) parked, awaiting user answer. Then EXOS→VOSS Socratic walk if time.
type: project
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Session Date
**2026-05-08** (Friday)

## Session goal
Reproduce the May 6 DHCP failure deterministically so we can:
1. Capture AP/EXOS state DURING the failure (not just after — yesterday's lesson: reboot consumed the evidence)
2. Validate Hypothesis (b) WPA2 key-state wedge with a controlled trigger
3. Build muscle memory on the diagnostic procedure

## Status as session opened (morning May 8)
- Yesterday's full DHCP work captured in `project_dhcp_macbook_incident.md` + EOD HTML live at `https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260507.html`
- Lab is in **post-reboot good state** (per yesterday's confirmation)
- User asked the deeper question: **"Why did TK/GTK corrupt in the first place?"** — that's the *trigger* layer below yesterday's *mechanism* layer

## Deeper trigger insight surfaced this morning
From the May 6 pcap data + show dhcp-server output:
- iPhone Corporate session had MAC `e2:68:dc:4d:37:3c` with lease 10.20.0.100
- MacBook's en0 capture in v5 also showed source MAC `e2:68:dc:4d:37:3c` — **same MAC value, two devices, same SSID**
- That is the trigger that the elimination matrix didn't surface yesterday. The (b) WPA2 key-state mechanism is downstream of this collision.

## Socratic Question Posed (PENDING USER ANSWER)
"Given two devices ended up with the same LAA MAC on the same SSID, what's the most likely failure cascade at the AP?"

Options offered:
| # | Hypothesis | Mechanism |
|---|-----------|-----------|
| (A) | Stale per-MAC key slot in AP's hardware crypto engine | iPhone's PTK still bound; MacBook joins, runs new 4-way; AP either evicts cleanly OR double-binds → encrypts with old TK while client decrypts with new → silent fail |
| (B) | GTK rotation triggered by perceived "duplicate client" | AP sees same MAC as already-associated client = anomaly; deauth-of-old + GTK rotation; Group Key Handshake to iPhone wedges → iPhone deaf to broadcasts |
| (C) | macOS dual-interface bonding | MacBook Ethernet on Port 5 + Wi-Fi joining; macOS aborted/restarted EAPOL mid-flight when wired preferred → AP left with half-installed TK |
| (D) | All three layered — A is direct cause for MacBook, B propagates to iPhone, C made handshake fragile |

User to commit to one before we proceed. (D) is my best guess given the symptom pattern (bidirectional silent + iPhone broke too) but they need to commit first per Socratic norm.

## Baseline-Capture Playbook (ready to execute)

### AP3000 commands (via XIQ → Device 360 → SSH/CLI)
- `show client / show client mac <MAC>` per device
- `show ssid <name> station`
- `show interface wifi0/wifi1 counter`
- `show counters wireless` (THE baseline counter for decrypt errors)
- `show roaming cache`
- `show interface wifi0 _per-chain / dfs _detail`
- `filter 1 l3 protocol 17 src-port 68 dst-port 67` (DHCP DISCOVER/REQ)
- `filter 2 l3 protocol 17 src-port 67 dst-port 68` (OFFER/ACK)
- `capture interface wifi0 duration 60 filter or 1`
- `capture save interface wifi0 dhcp_baseline_good.pcap`
- Optional 30s: `_kdebug wifi-driver wifi0 msglevel wsec` then `_debug stop`

### SW1 EXOS commands
- `show dhcp-server` (full + per-VLAN leases)
- `show fdb vlan VLAN_20_Corporate_Wireless / VLAN_30_Guest_Wireless / port 3`
- `show iparp vlan ...`
- `show ports utilization / rxerrors`
- `show log`
- `show vlan`

### MacBook (en0) commands
- Wireless Diagnostics → Capture (60s while connected + iPhone also up)
- `ifconfig en0`, `networksetup -getairportnetwork en0`, `sudo wdutil info`

### iPhone
- Tether through MacBook + capture on tethered interface, OR
- Sonoma Wireless Diagnostics → Sniffer mode (channel-locked OTA)

## Repro Strategies

### Option 1 — Spoof MAC (deterministic; tests Hypothesis A directly)
```
sudo ifconfig en0 ether <iphone-corp-mac>
# Then connect MacBook to Corporate with spoofed MAC while iPhone still connected
```
Should reproduce TK collision exactly. Watch decrypt-error counter spike on AP.

### Option 2 — Dual-interface toggle (replays original conditions; tests Hypothesis C)
```
# MacBook wired on Port 5 (lease active on VLAN 10)
# Repeatedly: System Settings → Wi-Fi OFF → ON
# Each cycle creates a new EAPOL race
```
Lower confidence trigger but matches May 6 conditions exactly.

## Pending Items — ALL COMPLETED
- [x] User commits A/B/C/D → **(D) layered cascade**
- [x] User picks repro strategy → **dual-interface toggle, no MAC spoof**
- [x] Lab state → **good, baseline-ready** (clients actually on Corporate, not Guest as initially thought)
- [x] Baseline capture run (partial — AP Eth0 capture started, ADSP blocked wifi captures, kdebug crashed SSH)
- [x] Trigger fired (en8 moved from QF Modem → SW1 Port 5)
- [x] Capture during failure (work MBP en0 pcap = 24MB, 596s, full failure cycle)
- [x] Diff captures → mechanism CONFIRMED via 3-client fingerprint (2 fail, immune = trigger device)
- [x] Write up findings — published as session_summary_20260508.html + session_log_20260508.md

## OUTCOME — Hypothesis B confirmed via repro
**The 3-client failure pattern is the smoking gun:**
- Work MBP (trigger device, UAA `84:2f:57:94:bd:cf`): NEVER FAILED
- Personal MBP (UAA `74:a6:cd:8b:19:60`): failed 4.3 min
- iPhone (LAA `62:75:85:f6:4e:3e`): failed 2.7 min

This is a unique fingerprint of GTK rotation propagation (B). Per-client TK collision (A) cannot produce 2-of-3 failures, and (A) is also ruled out because two of three clients use stable UAA MACs (no collision possible).

The trigger device is immune because it gets the new GTK_v2 first as the active client. Lagging clients miss the M1/M2 group key handshake → keep GTK_v1 → silent on broadcast traffic.

**Self-recovery in 2-4 minutes confirmed** — yesterday's reboot was unnecessary.

## REFINED TRIGGER MECHANISM (user discovery May 8 morning)
**The exact deterministic trigger:** moving the **work MacBook's Ethernet cable from QF Modem (DMZ side) to SW1 Port 5 (lab side)** — while the same MacBook's en0 Wi-Fi is associated to Guest_Wireless via AP1.

**iPhone's DHCP starts failing (169.254.x.x APIPA) the moment Port 5 link comes up. Disconnecting Port 5 = problem goes away.**

This is more specific than yesterday's "MacBook Wi-Fi joining" framing. The trigger isn't joining Wi-Fi — it's the **arrival of the wired interface on the same SW1 fabric** while Wi-Fi is already associated.

### Best-fit mechanism for the refined trigger
**(C) macOS dual-interface state change → triggers (B) GTK rotation propagation:**

1. Work MacBook is associated to Guest_Wireless (en0, VLAN 30)
2. Plug Ethernet (en8) into Port 5 (VLAN 10 wired) — link comes up
3. macOS detects new wired link, re-evaluates routing preference (wired > wireless)
4. macOS may issue: route reshuffle, gratuitous ARPs, possibly Wi-Fi disassoc/re-assoc, EAPOL re-key
5. AP perceives "client state change" event → triggers GTK rotation as security response
6. Group Key Handshake fires out to all OTHER clients (iPhone, personal MacBook) on Guest_Wireless
7. M1 (GTK_v2) → M2 (ACK) — under load / racing other state, M2 from iPhone is lost
8. AP commits GTK_v2; iPhone still on GTK_v1
9. Broadcast DHCP renewal traffic encrypted with GTK_v2 → iPhone can't decrypt → silent drop
10. iPhone's lease expires → APIPA 169.254.x.x

Key insight: **the trigger isn't a key collision (A); it's GTK rotation propagation (B) caused by the perceived dual-interface event (C).** Hypothesis A may not be involved at all in this scenario — it requires a specific LAA-MAC collision which the work MacBook's separate UAA wired MAC + LAA Wi-Fi MAC may not produce.

### Why this is so easily reproducible
- Work MacBook's wired and Wi-Fi interfaces have *different* MACs (en8 UAA vs en0 LAA), so this isn't a MAC flap on SW1
- BUT both interfaces belong to the same OS network stack, so macOS does a route preference re-evaluation
- That tickles the Wi-Fi state machine just enough to perturb the EAPOL key state
- Net effect: every time you plug in Port 5, GTK rotates, and the rekey races break iPhone

## Validation strategy
- Baseline capture (this section below) when ALL is working
- Trigger: plug Ethernet into Port 5
- Failure capture: same commands, same window
- Diff:
  - AP `show counters wireless` → expect `gtk_rekey_count` or similar to increment (B)
  - AP `_kdebug wifi-driver msglevel wsec` log → should show GTK rotation event
  - SW1 `show fdb port 5` → should show work MacBook MAC NEWLY learned (the trigger event timestamp)
  - iPhone DHCP capture → DISCOVERs going out, no OFFERs visible from the AP-encrypted broadcast direction

## MacBook Interface Map (corrected May 8 morning)
- `en0` = Wi-Fi (built-in)
- `en8` = Ethernet (USB-C adapter, Apple Silicon Mac assigns higher numbers because en1-en7 are internal/legacy)
- The original baseline command list had `en1` — corrected to `en8`. iPhone is N/A here (its single Wi-Fi interface is `en0` internally).

## Cross-references
- `project_dhcp_macbook_incident.md` — yesterday's full saga, mechanism diagnosed (b)
- `reference_dhcp_wifi_triage_runbook.md` — 5-stage triage
- `reference_ap3000_command_themes.md` — commands organized by lens
- `reference_exos_command_themes.md` — EXOS lens commands
- May 7 EOD HTML — Section 7 (Validation) + Section 8 (Triage runbook) + Section 8b (Lens framework)

## EXOS→VOSS Socratic walk — still parked for later today / tomorrow
Per yesterday's plan, after the DHCP repro we have:
1. Outline EXOS→VOSS 4 principles (memory: `reference_exos_voss_4_principles.md`)
2. Walk SW2 config Socratically (NLLM verbatim recipe deliberately NOT cached; reference docx for sanity-check only)
3. Open questions to resolve: IPFIRE = IPE? Port 1/2 vs 1/3 for AP2? IPE LAN1 IP = 192.168.0.25 (NLLM hypothesis, verify on switch)