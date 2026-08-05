---
name: AP Session June 5 2026 — AH-556680 CLI Diagnostics
description: Full AP SSH session dialogue June 5 2026. Script logging, channel width fix, CRC error drop, PPSK VLAN 30 root cause, dynamic channel width. For EOD HTML.
type: project
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
## AP Details
- Hostname: AH-556680
- IP: 192.168.0.12
- Platform: Extreme Networks / Aerohive HiveOS, WiFi 6 (11ax), dual-band only (no 6GHz)
- Radios: wifi0 = 2.4GHz, wifi1 = 5GHz
- Connected to XIQ org 554341 via CAPWAP + DTLS (oh-cws-3.extremecloudiq.com)

## Session Log Files
- `/Users/khukhan/Documents/AP_Sessions/ap_session_20260605_150856.txt` — Session 1
- `/Users/khukhan/Documents/AP_Sessions/ap_session_20260605_152939.txt` — Session 2

## Script Logging Setup
- Directory created: `~/Documents/AP_Sessions/`
- Command: `script ~/Documents/AP_Sessions/ap_session_$(date +%Y%m%d_%H%M%S).txt`
- SSH in, do work, `exit` (ends SSH), `exit` again (ends script recording)
- `Script done` message confirms file saved
- macOS: `watch` not built-in — use `while true; do ...; sleep 2; done` loop instead
- `sshpass` needed for passwordless loop; SSH key auth is cleaner long-term

## VERBATIM USER DIALOGUE (keep for EOD HTML)

"I want to study this file... pls review" [ap_session_20260605_150856.txt]

"Also, I do need to check why PPSK is falling back on VLAN_30."

"Can you tell me how to figure out why the client devices are not selecting 40MHz channels?"

[After session 2 review]

"How can I change 80MHz via CLI on AP?"

"What is the enable command on AP?"

"Radio profile radio_ng_11ax-5g already exists! / channel-width 80 ^-- unknown keyword"

"Confirmed it changed. Now for the commands to see if it helped with the CRC?"

"7% CRC error, can I look at it live?" [was 34.34% before]

[watch command failed — not on macOS]

"does not work because it asks for password every time"

"anyway 7% CRC" [accepted result]

"What is my noise figure and noise floor?"

"No, because on the UL I am NOT asking for anything :)" [re: 24M uplink = no upstream traffic, not a real measurement]

"No need. Now, this AP change, will it be saved? How to make sure that EP1 policy has it at 80MHz which I pushed? Can it not be dynamically chosen by RRM?"

"Can I not push 'dynamic channel width=enabled' via CLI?"

"radio profile radio_ng_11ax-5g dynamic-channel-width enable worked"

"Enabled. Verified. So, how can I now test what the negotiated bandwidth maybe? Can I observe this in real time to see? Can I do this for 2.4GHz or 6GHz?"

"Make sure you are saving ALL this DIALOGUE in MEMORY for EOD HTML PLS."

## Key Findings — Session 1 (150856)

### Config Summary
- SSIDs: WC_Seattle-PPSK (WPA2+PPSK, external XIQ) + WC_Seattle_Guest (Open+CWP)
- User profiles: WC_TeamUSA→VLAN10 (attr53), WC_TeamAUS→VLAN20 (attr54), WC_Seattle_Guest1→VLAN30 (attr50)
- 2 stations on wifi1.1 (5GHz Ch149 VLAN30): 74a6:cd8b:1960 (10.30.0.101) + 867d:3eb3:90ec (10.30.0.103)
- Both: UPID=50, SNR 53-57dB, TX 162-286Mbps (AP→client), RX ~24Mbps (client→AP), 20MHz, no MU-MIMO

### Issues Found
1. `show stations` failed — correct is `show station` (no 's')
2. Both clients on VLAN 30 — PPSK maps to Guest group not Team groups
3. RX rate 24Mbps — no upstream traffic being generated (explained later)
4. 20MHz channel width — hardcoded in radio profile
5. Sensitive values in log: CWP api-key, api-nonce, telegraf token — keep file private

## Key Findings — Session 2 (152939)

### Channel Width Root Cause
- `show interface wifi1` → `Channel width=20MHz; Dynamic channel width=disabled`
- `show radio profile radio_ng_11ax-5g` → `Channel width=20MHz` — hardcoded in profile
- ACSP shows Ch149 cost=0 (completely clean), no reason to be at 20MHz
- WiFi 6 features all disabled: OFDMA DL/UL, MU-MIMO, TWT, BSS Color

### PPSK VLAN 30 Root Cause — CONFIRMED
- `show auth interface wifi1.1` revealed: both clients have **same PMK hash (0360\*)** — using identical shared PPSK
- AP-side user-profile-policy is correct (3 rules, correct attr IDs)
- CAPWAP healthy: Connected securely, 0.01% keepalive loss
- Root cause: PPSK assigned to WC_Seattle_Guest1 group in XIQ — fix is in XIQ not AP
- Fix: go to XIQ → Users → PPSK Users, reassign PPSK to correct team group

### CRC Error Fix
- Before: 34.34% CRC error airtime, Summary state=High collision
- After 80MHz: 7% CRC — massive improvement
- Noise floor: -90dBm (excellent, near thermal noise)
- Remaining 7% likely hidden node / collision, not RF noise

### Commands Applied
- `radio profile radio_ng_11ax-5g channel-width 80` → confirmed working
- `radio profile radio_ng_11ax-5g dynamic-channel-width enable` → confirmed working
- `save config` after each change

### HiveOS CLI Notes
- No `enable` command — SSH lands directly at # (privileged) prompt
- No sub-context mode — full command on one line: `radio profile <name> <param> <value>`
- `show station <mac> counter` (not `detail`) for per-client stats
- `show acsp channel-info` (not `show acsp interface wifi1`)
- `show radsec proxy status` (not just `show radsec`)
- `show high-density profile <name>` (not just `show high-density`)

## Noise Floor
- Noise floor: -90dBm (excellent)
- Running avg: -93dBm
- Short term: -89dBm
- Client RSSI: -42dBm → SNR = 48dB (excellent, supports MCS10/11)

## 24Mbps Uplink Explanation
- Not a real measurement — no upstream traffic being generated from clients
- 24Mbps = ACK/keepalive minimum rate only
- Need iperf/speed test from client to measure real uplink capability

## Permanent Fix Required
- CLI changes saved to AP flash (save config) — survive reboot
- BUT XIQ config push will overwrite → must update EP1 radio profile too
- EP1: Configure → Radio Profiles → radio_ng_11ax-5g → Channel Width = 80MHz + Dynamic = enabled

## Real-Time Monitoring Commands
- `show station` — Chan-width column shows negotiated width per client
- `show interface wifi1 | include CRC` — CRC error rate
- `show interface wifi1 | include Noise` — noise floor live
- `show interface wifi0` — 2.4GHz equivalent
- No 6GHz — AP is WiFi 6 (11ax) only, not WiFi 7. 6GHz requires AP5060 or similar WiFi 7 AP.

## Why: Key Decisions
- Channel width fixed in radio profile (AP-side issue confirmed, not clients)
- Dynamic channel width better than hardcoded 80MHz — ACSP/RRM auto-adjusts
- PPSK fix must happen in XIQ — AP policy is correct
