---
name: EOD June 2 2026 — 802.11 Auth/Assoc/Roaming study session
description: Reference to June 2 study EOD covering PSK/EAP-PEAP/EAP-TLS/4-way handshake/802.11r FT roaming with pcap library
type: reference
originSessionId: 4d3a8b8b-a7df-4354-a376-1ed6b280263d
---
EOD HTML live at: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/docs/session_summary_20260602_wifi_auth_roaming.html

Local file: `/Users/khukhan/5320-onboarding-agent/docs/session_summary_20260602_wifi_auth_roaming.html`

**Why:** Deep-dive study session on 802.11 auth/assoc/roaming — PSK vs EAP-PEAP vs EAP-TLS, 4-way handshake internals (PMK→PTK), 802.11r Fast Transition roaming proof. Linked from all 3 index pages (commit 9603025).

**How to apply:** Reference this when continuing 802.11 security/roaming study or building on this knowledge in future sessions.

## pcap library (5 files)
All at `docs/data/pcaps/` in 5320-onboarding-agent repo. Downloadable from the EOD page.

| File | What it shows |
|------|--------------|
| `wpa2-ft-psk.pcapng` | Best roaming file — Auth→Assoc→4-way→data→FT Auth→Reassoc (no EAPOL on roam). Roam at frame 24 t=62.8s. AP1→AP2. |
| `wpa2-ft-eap.pcapng` | EAP-PEAP (TLS+MSCHAPv2) + 4-way + 802.11r FT structure |
| `wpa-eap-tls-dot1x.pcap` | Full EAP-TLS mutual cert exchange + 4-way (frames 22–25) |
| `wpa-4way-handshake.pcap` | Clean WPA2 PSK 4-way only. Filter: `eapol` |
| `wpa3-ft-roaming.pcapng` | Limited — beacons + probes only, no actual roam |

## Shell capture aliases (~/.zshrc)
- `wcap <seconds> <name>` — timed capture on en0 → saves to Desktop → opens Wireshark
- `wscan` — live stream en0 → Wireshark (no file)
- Both capture Ethernet layer only (no 802.11 radio headers — no admin/sudo on work Mac)
