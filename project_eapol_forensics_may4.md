---
name: EAPOL & Packet Forensics — Horizon Lab Capture (May 4 2026)
description: 306K-frame Wireshark deep dive on Horizon lab. WPA2 vs WPA3 key hierarchy, 4-way handshake forensics, rogue NAV attack found. EOD HTML in Downloads, NOT yet pushed to GitHub Pages.
type: project
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Session Date
**2026-05-04** — fills the May 1 → May 7 memory gap.

## EOD HTML
- Original: `/Users/khukhan/Downloads/EOD_Blueprint_May4_2026.html`
- **Repo copy:** `/Users/khukhan/5320-onboarding-agent/docs/session_summary_20260504.html`
- Title: "EOD Blueprint — May 4 2026 | EAPOL & Packet Forensics"
- Status: ✅ **Pushed May 7** — commit `b9b4029` on `feature/auto-deploy-agent` (origin synced)
- GitHub: https://github.com/khursheedkhanaiforgood-ai/5320-onboarding/blob/feature/auto-deploy-agent/docs/session_summary_20260504.html
- ⚠️ Not yet on `gh-pages` (live URL pending merge) and `index.html` was NOT updated — matches the Apr 30 EOD pattern in this repo (per-session EODs added to docs/ but not auto-linked from index).

## Capture Stats
| Metric | Value |
|--------|-------|
| Total Frames | 306K |
| Unique Clients | 24 |
| CTS Response Rate | 44.6% (warn) |
| Rogue MAC Frames | 2,504 (danger) |
| 4-Way Handshake Time | 14ms |
| SSID Security | WPA2 |

## Concepts Mastered (11)
1. **PMK Origin Paths** — WPA2-PSK: PBKDF2(password) = static PMK. WPA3-SAE: DH exchange = ephemeral PMK. No RADIUS = PSK path.
2. **PTK Derivation** — PRF(PMK, label, AA, SPA, ANonce, SNonce) → KCK + KEK + TK. All inputs cleartext except PMK.
3. **MIC Purpose** — Integrity only, not encryption. M2 MIC proves client knows PMK. Computed with KCK.
4. **KEK vs TK** — KEK encrypts GTK inside M3 before TK is installed. Install bit = load TK into hardware crypto engine.
5. **GTK Impact** — Group key decrypts ALL broadcast/multicast. Cracking GTK reveals ARP, DHCP, mDNS — full VLAN topology.
6. **Replay Counter** — Prevents KRACK. MIC = integrity, Replay Counter = freshness. Both required.
7. **Forward Secrecy** — WPA3-SAE: DH ephemeral keys discarded after session. Password leak cannot decrypt past captures.
8. **NAV / Virtual Carrier Sense** — Duration field in every frame. RTS sets NAV, CTS decrements. CTS-to-Self Duration=0 resets all timers.
9. **RTS/CTS Directionality** — Either side initiates. AP sends 70% of RTS in this capture (heavy downlink to 24 clients).
10. **Monitor Mode vs AP Capture** — RSSI on AP TX frames = third-party sniffer. All frames show receive signal level.
11. **Rogue Device Fingerprinting** — Unknown OUI + locally-administered MAC bit + single frame type + no association = NAV attack signature.

## Rogue Device Found
- MAC: **e0:3e:44:10:aa:64**
- Signature: NAV attack (CTS-to-Self with bogus duration)
- 2,504 frames flagged

## Why this matters
This session was a Socratic deep-dive on 802.11 frame forensics using the live Horizon Custom Fabrication lab capture — establishes the L1/L2 baseline for the WiFi Digital Twin engineering work.

## Next-Step Checklist (from May 4 EOD — Panel 8)

### P1 (Immediate — SW2 VOSS)
1. Confirm `router ?` syntax in config-dhcp mode on SW2
2. Run `show ip interface` + `show ip arp vrf GlobalRouter` — verify VLAN 4047 on Port 1/1
3. Find IPE lan1 IP via IPE Edit Configuration — required before SW2 default route
4. Safe CLI block: `manual-area 49.b0b1` → `vlan create 70/80` → `vlan i-sid 100070/100080` → `fa enable Port1/3`

### P2
- DHCP config for VLAN 70 (Corp_New) and VLAN 80 (Guest_New) on SW2
- IPE transit config — **GRE tunnel SW2 → RDU Raleigh** (resolves Q2 from May 1 memory: tunnel = GRE)
- VIQ policy — Corp/Guest SSIDs on VLAN 70/80 for AP2 via Fabric Attach

### P3
- ACL GUEST_ISOLATION verification on BOTH switches (Anycast Gateway — SW2 routes locally)

### Deep-Dive Queue
- OWE (Opportunistic Wireless Encryption) deep dive
- Packet capture practicum — SPAN port on SW1, decode encrypted frames with known PTK
- 802.1X wired port auth — EAP-TLS vs PEAP on SW1 Port 5
- WiFi L1 PHY — OFDM subcarriers, guard intervals, MCS table, spatial streams
- KRACK attack lab — reproduce replay counter vulnerability in controlled env

## Cross-References
- Updates Q2 from `project_5320_new_arch_voss_ipe.md` (RDC tunnel = GRE, was OPEN)
- Pending tasks merged into `project_session_20260507.md`
