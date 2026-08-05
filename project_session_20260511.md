---
name: Session May 11 2026 — EXOS→VOSS + 802.1X Learning
description: EXOS→VOSS 4 principles mapped to SW2 live config. Full 802.1X PEAP-MSCHAPv2 learning arc. VLAN 153/154 architecture confirmed. Capture checklist ready for tomorrow's NPS deployment.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Session Date
**2026-05-11** (Monday, 2:20–4:20 pm Pacific)

## What Was Covered

### EXOS→VOSS 4 Principles — SW2 Live Config (KhKLab-SW-01, FE 9.3.2.0)
- **Brain (IS-IS)**: ip-source-address 10.159.4.1 (Loopback 1) = SW2 fabric identity; is-type l1; redistribute direct → isis apply redistribute direct
- **Muscle (SPB)**: spbm ethertype 0x8100; nick-name 0.00.02; multicast enable; B-VLANs 4051/4052 auto-created
- **Service (I-SID)**: VLAN 153→I-SID 144153, VLAN 154→I-SID 144154, VLAN 155→I-SID 144155, VLAN 156→I-SID 144156, VLAN 4048→I-SID 15999999 (onboarding). BOBKit-4- prefix = XIQ SiteEngine BOB workflow
- **Edge (FA/Auto-Sense)**: All 20 ports auto-sense enable. Rules: wap-type1→144153, camera→144155, proxy-no-auth→144156, data→144156, onboarding→15999999. Discovery is LLDP FA TLV based — NOT MAC based.

### Conceptual Corrections Locked In
- IS-IS = Brain (NOT service). I-SID = Service.
- FA discovery = LLDP TLV, not MAC address
- SW2 is DHCP server (not relay to external). Self-relay: VLAN SVI → management CLIP 10.158.4.1. 24h leases.

### 802.1X Full Learning Arc — PEAP-MSCHAPv2
- Three actors: Supplicant (MacBook), Authenticator (AP2), Auth Server (NPS)
- VLAN assignment = NPS Network Policy decision (Attr 81 = "154"), NOT derived from credentials
- MSK generated from TLS master secret + MSCHAPv2 key material. PMK = first 256 bits of MSK (unique per session)
- 4-way handshake IDENTICAL to WPA-Personal — only PMK source differs
- Username plaintext in EAP-Response/Identity (last visible frame before PEAP tunnel)
- Auth success signals: EAP-Success → M1/M2/M3/M4 4-way → DHCP from 10.154.4.x

## CRITICAL — VLAN 153/154 Architecture (confirmed by lab owner May 11)
- **VLAN 153 = AP Management = UNTAGGED (native)**. AP2 sends management frames untagged. SW2 Auto-Sense receives untagged → maps to I-SID 144153. DO NOT TOUCH in XIQ.
- **VLAN 154 = Wireless Users = TAGGED**. AP2 tags user frames with VLAN 154 after 802.1X auth (RADIUS Attr 81 = "154"). Configure this in XIQ tomorrow.

## Tomorrow's XIQ Deployment (when NPS is up)
1. Create 802.1X SSID → security WPA2-Enterprise → user VLAN = **154** (tagged)
2. Add NPS as RADIUS server (get IP + shared secret from boss)
3. Do NOT touch AP management VLAN (153/native — leave as-is)
4. Push to AP2

## Capture Checklist for Tomorrow
| # | Where | Tool | Signal |
|---|-------|------|--------|
| 1 | OTA | Wireless Diagnostics sniffer | Username in EAP-Response/Identity → EAP-Success → M1/M2/M3/M4 |
| 2 | en0 | Wireshark | DHCP OFFER from 10.154.4.1, client gets 10.154.4.x |
| 3 | NPS | Event Viewer → Security | Event 6272 = granted, 6273 = denied |
| 4 | SW2 | show fdb vlan 154 | Client MAC on port 3 after auth |
| 4b | SW2 | show vlan members port 1/3 | VLAN 154 added alongside 153 |

## Sprint Backlog
- QoS ACL fix → week of May 18 (branch feature/qos-egress-validation)
- AP1_Data download (QoS/DHCP pcaps) → week of May 18
- WiFi Digital Twin Sprint 1 → week of June 1

## EOD
- `session_summary_20260511.html` live at GitHub Pages (commit 45ef275 on main)
- `index-nyt.html` updated with May 11 card + Simulators section (3 cards)
- SW2 running config embedded in EOD HTML as collapsible section

## dot1x_simulator.html — COMPLETE (commit 58a6bc3)
- 17-step PEAP-MSCHAPv2 interactive simulator, dark theme
- **Play/Pause** auto-advance with speed selector (Slow 2s / Normal 1.2s / Fast 0.5s)
- **Restart bug fixed**: history{} arrays now cleared on reset (was retaining old messages)
- Copyright footer added
- Simulators section on index-nyt.html links all 3: WPA (Claude artifact) + 802.11be (Claude artifact) + 802.1X (dot1x_simulator.html)
