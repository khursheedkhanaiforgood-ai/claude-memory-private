---
name: DHCP MacBook Incident — May 6 lab failure, root cause TBD
description: MacBook fails DHCP on BOTH Corp and Guest SSIDs while iPhone works fine. Reboot APs+MacBook fixed it. Apple per-SSID privacy MAC documented. Alex/NotebookLLM "MAC flap" hypothesis rejected. Investigation pending — prior-Claude trail points at AP3000 forwarding state.
type: project
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Date / Setup
- **Incident:** 2026-05-06 evening
- **Recovery:** Reboot of APs + MacBook (root cause not yet determined)
- **Lab:** EXOS_SW1 (192.168.0.28) — Port 1=QF Modem, Port 3=AP1 (AP3000), Port 5=MacBook wired
- **VLANs:** 10 (Wired Corp 10.10.0.x), 20 (Wi-Fi Corp 10.20.0.x), 30 (Wi-Fi Guest 10.30.0.x)
- **NOT involved:** SW2 VOSS / IPE — Port 10 was unplugged during this incident. Pure SW1/EXOS issue.

## Symptom Timeline
| T | Event |
|---|---|
| T0 | iPhone working fine on Wi-Fi Corp + Guest |
| T1 | User turns on MacBook Wi-Fi while MacBook also wired on Port 5 |
| T2 | iPhone DHCP breaks; MacBook also can't get DHCP |
| T3 | Pcap captured from MacBook en0: 7 DHCP DISCOVERs over 50s, ZERO server replies |
| T4 | Reboot APs + MacBook → all working again |

## Pcap Files
| File | SSID | MacBook MAC | DISCOVERs | Replies |
|------|------|-------------|-----------|---------|
| `/Users/khukhan/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Desktop/pcap_May6_v4.pcapng` | Guest | `4e:60:81:bb:23:18` | 7 | 0 (one IGMP from 192.168.1.1 = QF modem) |
| `/Users/khukhan/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Desktop/pcap_May6_v5.pcapng` | Corporate | `e2:68:dc:4d:37:3c` | 7 | 0 (completely silent) |
| `/Users/khukhan/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Desktop/WiFi_May7_10:40am.pcapng` | Corp (post-reboot) | — | — | 299 frames, full TCP/TLS/HTTP/DNS — working |
- Tshark binary: `/Applications/Wireshark.app/Contents/MacOS/tshark`
- Prior dialogue (Claude May 6): `/Users/khukhan/Downloads/DHCP_Issue_Exchange_Claude_May 6 2026.docx`

## Tech-Support Dumps (May 7 — captured post-incident)
| File | Source | Captured | Size | Notes |
|------|--------|----------|------|-------|
| `tech_support_SW1_EXOS_May7_2026.txt` | EXOS `show tech-support` on 5320-16P | 10:08:06 EDT | 1.07 MB | full switch state — VLANs, DHCP server, FDB, ARP, log |
| `tech_support_AP1_May7_2026.txt` | AP3000 `show tech` (XIQ pulled) | 08:28:59 EDT | 444 KB | debug + radio + wipsd logs — should contain decrypt counters |
**Live URLs (gh-pages):**
- https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may7-dhcp/tech_support_SW1_EXOS_May7_2026.txt
- https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may7-dhcp/tech_support_AP1_May7_2026.txt
**Source paths (Downloads):**
- `/Users/khukhan/Downloads/device_support_logs_2026-May-07-18_07_45_EXOS_SW1/07052026_1808` (SW1)
- `/Users/khukhan/Downloads/device_support_logs_2026-May-07-15_28_59_AP1/core_dump41 2/show_tech_result copy_AP1_May 7 2026.txt` (AP1)

## Three MacBook MACs (Apple per-SSID privacy randomization)
| Interface | MAC | UAA/LAA | Source |
|-----------|-----|---------|--------|
| Wired (Port 5, VLAN 10) | `80:69:1a:dd:0f:c3` | UAA — Apple OUI burned-in | show dhcp-server |
| Wi-Fi Corporate (VLAN 20) | `e2:68:dc:4d:37:3c` | LAA — privacy randomized | pcap_May6_v5 + show dhcp-server |
| Wi-Fi Guest (VLAN 30) | `4e:60:81:bb:23:18` | LAA — privacy randomized | pcap_May6_v4 |

iPhone MAC on Corporate (working): `62:75:85:f6:4e:3e` (LAA, lease 10.20.0.101).

## Competing Hypotheses
### Alex (NotebookLLM, May 7) — "MAC Flap between Port 5 and Port 3"
**Premise:** Same MAC appears on both wired and Wi-Fi → SW1 FDB locks → DHCP listener stalls.
**Verdict: REJECTED.** Three different MACs (UAA wired, LAA per-SSID Wi-Fi). No physical interface shares a MAC with another. Nothing flaps. Foundation wrong → all 3 of Alex's Socratic Qs invalid.

### Alex Q1 — "Wrong VLAN landing (192.168.1.1 modem)"
**Verdict: PARTIAL.** Only fits v4 (Guest pcap, has 192.168.1.1 IGMP). v5 (Corporate pcap) is completely silent — no 192.168.1.1, no IGMP, no traffic at all. Theory only explains one of two failure modes.

### Prior Claude (May 6) — "Soft state in AP3000 forwarding"
**Status: TRAIL.** Got to: SW1's FDB on VLAN 20 had ONLY iPhone MAC (`62:75:85:f6:4e:3e`). MacBook's `e2:68:dc:4d:37:3c` was NEVER in FDB even though MacBook was actively transmitting DISCOVERs. Frames never reached SW1 Port 3. **Reboot fixing it = volatile in-memory state cleared.** Most likely culprit: AP3000's internal MAC bridge / forwarding engine wedged for the MacBook's Wi-Fi MAC specifically. Boss's XIQ "buffers and forwarding engines" investigation aligns with this.

### CONFIRMED ANSWER (May 7 Socratic) — WPA2 Key State (b)
4-table elimination matrix (Apple per-SSID privacy + Wi-Fi internals):

| Option | Direction | Selectivity | Silent? | Verdict |
|---|---|---|---|---|
| (a) Client assoc state | Both | Per-client | Sometimes (logs) | Runner-up |
| **(b) WPA2 key state** | **Both** | **Per-client (TK) / All-clients (GTK)** | **Yes** | ✅ **Best fit** |
| (c) Bridge FDB | Forwarding only | Per-MAC, floods unknowns | No (floods) | Fails: floods ≠ silent drops |
| (d) TX queue | **Downstream only** | Per-client or radio-wide | Sometimes | Fails: can't explain upstream loss |

**(c) eliminated:** Linux bridge FDB *floods* unknown unicast — doesn't drop. SW1 would have seen flooded DISCOVERs and learned MAC. It didn't → frames dropped UPSTREAM of FDB lookup → before bridge → at decryption layer.

**(d) eliminated:** TX queue is downstream-only. SW1 FDB never saw upstream DISCOVERs → upstream is also broken → can't be queue alone.

**(b) wins:** TK install for one client = bidirectional silent failure for that MAC. Reboot clears via re-handshake. **Tie-in to May 4 EAPOL EOD: Concept #4 (KEK vs TK install bit) is the exact mechanism.**

### TWO-KEY EXTENSION (May 7 plot twist) — iPhone ALSO failed
User reported iPhone DHCP also stopped working AFTER MacBook tried to join. This breaks single-client-TK theory but fits **GTK rotation wedge**:

- **TK** = per-client unicast key (4-way handshake). One client's TK going bad = one client deaf.
- **GTK** = group/broadcast key (Group Key Handshake, M3 of 4-way + periodic refresh). One broken GTK = ALL clients deaf to broadcasts.

**Likely sequence:**
1. MacBook 4-way handshake wedged or timed out → MacBook deauth'd
2. AP rotates GTK on disassoc (security: prevent old client from decrypting future broadcasts)
3. Group Key Handshake to iPhone wedges (M1 GTK_v2 sent, M2 ACK lost)
4. AP now encrypts broadcasts with GTK_v2; iPhone still on GTK_v1
5. DHCP OFFER (broadcast frame from server) encrypted with GTK_v2 → iPhone can't decrypt → silent
6. Reboot of AP wipes all key state, full re-handshake of all clients with fresh GTK

**This is the full answer:** TK wedge for MacBook + GTK rotation wedge for iPhone, both triggered by MacBook's failed join. Tie-in: May 4 EOD Concept #5 (GTK Impact: "Group key decrypts ALL broadcast/multicast").

## Diagnostic Procedure (Next Time, BEFORE Reboot)
Capture FROM AP3000 via XIQ → Device 360 → Tools → SSH/CLI:
1. `show client` — confirm MacBook is listed; check state column (Authenticated / 4WAY-Done / Data-OK or stuck at 4WAY-PENDING)
2. `show client mac <MAC>` — per-client deep dive: TK Installed, cipher, frame counters
3. `show counters wireless` (or `show wlan-statistics`) — **decrypt errors / MIC failures** counter (THE smoking gun)
4. `show interface wifi0 statistics` — RX OK / RX errors / Decrypt failures
5. AP packet capture, filter `wlan.addr == <MAC>` — raw 802.11 + decryption result
6. Run twice 30s apart — diff the counters to see live failure rate
7. THEN reboot — compare counters/state after reboot to confirm clean re-handshake

**Key counter:** decrypt errors / MIC failures incrementing during failure window = proof of (b).

## Three-layer framing for "where did the frames vanish"
1. RF / 802.11 association (over-the-air leg) — reboot doesn't clear; iPhone passes same path → eliminated
2. **AP3000 internal forwarding** (RX radio → TX wired Ethernet bridge) — reboot CLEARS this; MacBook-specific failure → **leading hypothesis**
3. AP→SW1 wire (Port 3 link, VLAN tag, STP) — iPhone passes same wire → eliminated

## REPRO COMPLETED (2026-05-08) — Hypothesis B confirmed via fingerprint
**Trigger refined**: not "MacBook Wi-Fi joining" abstractly, but specifically:
**"Move work MacBook's en8 (USB-C Ethernet) from QF Modem to SW1 Port 5, while same MacBook's en0 is associated to Corporate via AP1."**

**Pcap captured**: `docs/data/may8-dhcp-repro/pcap_May8_workmbp_en0_repro.pcapng` (24 MB, 596s, work MBP en0). Trigger fired ~T+180-192s. Two clients failed:
- iPhone (`62:75:85:f6:4e:3e`, LAA): failed for 2.7 min, recovered T+163s post-trigger via gratuitous ARP at frame 12036
- Personal MBP (`74:a6:cd:8b:19:60`, UAA): failed for 4.3 min, recovered T+256s post-trigger via gratuitous ARP at frame 22098
- Work MBP (`84:2f:57:94:bd:cf`, trigger device): NEVER FAILED — immune to its own cascade

**Fingerprint = (B) GTK rotation propagation:** 2 of 3 clients fail; immune client is trigger device. (A) per-client TK collision would only break 1 client → doesn't match. (A) also requires LAA collision; today's repro has only iPhone using LAA (work + personal MBP both UAA hardware) → (A) ruled out for this scenario. (B) GTK rotation = trigger device gets GTK_v2 first as active client; lagging clients miss M2 ACK → keep GTK_v1 → silent on broadcast.

**Self-recovery confirmed**: en8 stayed plugged into Port 5 throughout. Recovery via periodic GTK refresh timer, client re-assoc on lease expiry, or AP auto-retry GTK push. Yesterday's reboot was unnecessary — would have self-healed in 2-4 minutes.

**Operational lessons**:
1. Never enable streaming kdebug on only SSH path — `_kdebug wifi-driver msglevel wsec` flooded SSH on busy WPA3 BSSID, dropped session
2. Use XIQ Device 360 → Tools → SSH/CLI for kdebug (cloud-relay path)
3. Wait 5 min before rebooting on next incident
4. Trigger-device-immune pattern is diagnostic for GTK desync
5. ADSP sensor mode blocks `capture interface wifi*` — use Eth0 instead

**Live URLs**:
- May 8 EOD: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260508.html
- Repro pcap: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may8-dhcp-repro/pcap_May8_workmbp_en0_repro.pcapng
- Session log (annotated summary): https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_log_20260508.md
- **Full verbatim dialogue DOCX (canonical)**: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may8-dhcp-repro/ClaudeAI_SessionDetails_May8_1209PST.docx (3,076 paragraphs spanning May 7 NYT rebuild → May 8 repro closure)
- **Full dialogue PDF (alternate format)**: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may8-dhcp-repro/Lab_Journal_DHCP_Repro_Dialogue_May8_2026.pdf

**Source-of-truth distinction:** The .md is an annotated summary (Claude-curated). The **DOCX is the canonical verbatim transcript** (user's export — 3,076 paragraphs of real conversation). The PDF is a shorter/alternate-format export. When debating what was actually said vs what was synthesized, **the DOCX is the canonical source**.

## Current State (as of 2026-05-08 — REPRO COMPLETED, DHCP work CLOSED)
- Lab is operational again (post-reboot, May 7 10:40 + en0 validation pcaps show full traffic)
- Root cause **DIAGNOSED** by elimination AND **VALIDATED** by ground-truth before/after pcap comparison
- **Answer: WPA2 key-state wedge — TK for MacBook + GTK rotation for iPhone**
- User completed Socratic walk: eliminated (a/c/d), arrived at (b) via mechanism reasoning, then proven by ground-truth
- **EOD live + linked from landing page (May 7 evening, commit `dcccb27` on main):**
  - https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_20260507.html
  - Linked from all 3 landing pages (index.html, index-nyt.html, index-harpers.html) as red EOD card "May 7 ★ DHCP"
  - All `data/may7-dhcp/` artifacts (pcaps, tech-support dumps, dialogue) live + clickable from EOD page
  - Initial 404 root cause: Pages serves from `main`, files were only on `gh-pages` — fixed by cherry-pick. See `reference_5320_repo_pages_config.md`.

### GROUND-TRUTH VALIDATION (the smoking gun)
The same Guest LAA MAC `4e:60:81:bb:23:18` appears in both:
- `pcap_May6_v4_guest.pcapng` (BAD): 7 DISCOVERs in 50s, 0 OFFERs (silent)
- `pcap_May7_en0_validation.pcapng` (GOOD): 1 DISCOVER → 1 OFFER from SW1 (10.30.0.x) → 1 REQUEST → 1 ACK in <1s

**All variables identical:** SSID, MAC, AP, switch, DHCP server, encryption protocol.
**Only thing that changed:** AP's per-client WPA2 key state (cleared by reboot, refreshed via fresh 4-way handshake).
**Conclusion:** This is Step 4 (Validation) of the Symptom→Hypothesis→Test→Validation framework — bidirectional silent failure cleared by re-handshake = WPA2 TK install state. **Confirmed by ground-truth comparison, not just elimination.**

### Validation Pcap added
- Source: `/Users/khukhan/Library/CloudStorage/OneDrive-ExtremeNetworks,Inc/Desktop/WiFi_en0_May7_2026.pcapng`
- Repo path: `docs/data/may7-dhcp/pcap_May7_en0_validation.pcapng`
- Live URL: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may7-dhcp/pcap_May7_en0_validation.pcapng
- 454 frames total, 18 DHCP frames, 4 full DORA cycles (Corp + Guest, both MACs)

## Open Question / Next Step
For next time the issue recurs (BEFORE reboot), capture from AP3000:
- AP CLI bridge/FDB table (does AP have MacBook's Wi-Fi MAC in its own bridge?)
- AP forwarding engine stats (drops, buffer pool exhaustion)
- AP-to-Port-3 trunk frame counters (TX from AP matches RX on SW1?)
- This is what XIQ AP-side data dump captures — link the boss's tool to the diagnostic question.

## Why this matters for understanding
**Apple per-SSID privacy MAC** is now a permanent fact of any wireless lab work. Switch DHCP tables and FDBs will be polluted with stale randomized MACs across reconnects. Don't troubleshoot by MAC alone — always cross-reference UAA/LAA bit and SSID context.
