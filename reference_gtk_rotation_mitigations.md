---
name: WPA3 GTK Rotation Cascade — Mitigation Levers
description: Layered mitigations for the May 6/8 GTK rotation cascade observed on AP3000 + WPA3-SAE-PMF. Ranked by impact. For engineering escalation to Extreme GTAC + R&D.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Context
Reproducible cascade where introducing a wired interface from a Wi-Fi-associated client triggers GTK rotation on the AP; lagging (idle) clients miss the Group Key Handshake re-sync and stay offline 2-4 minutes before self-recovery.

Source incident: `project_dhcp_macbook_incident.md`. Repro on 2026-05-08 confirmed Hypothesis B via 3-client fingerprint (2 fail, immune = trigger device).

## Why offline duration is 2-4 min
Failure clears when ANY of these fire:
- AP retries Group Key Handshake (M1) to lagging client (~60s default retry)
- AP rotates GTK on its periodic schedule (1-24h default)
- Client's DHCP lease expires → tries renewal → fails → re-associates (lease/2 timing)
- Client roams or naturally re-associates
- Client app activity triggers re-keying

**Failure duration scales inversely with client activity.** Most idle = longest miss window. Heavy active = stays in sync.

## Levers ranked by impact

### 🟢 Highest leverage — AP-side (XIQ)
| Setting | Default | Suggested |
|---------|---------|-----------|
| **Group Key Update Interval** | ≥1 hour | **5-15 min** |
| EAPOL retransmit timer | ~1s × 4 | 0.5s × 6 |
| Group Key Handshake retry | 60s+ | 30s |
| **802.11r Fast Transition + PMK caching** | varies | **enable** |

XIQ path: Network Policies → [policy] → Wireless Networks → SSID → Security → Advanced.

### 🟡 Medium leverage — DHCP
- **Increase DHCP lease time on SW1** from 7200s default → **86400s (24h)**: clients don't try to renew during the failure window
- Configure DHCP unicast (if supported): bypasses GTK encryption for OFFER/ACK

### 🟡 Medium leverage — proactive client recovery
Toggle Wi-Fi off → on on lagging client = forced fresh 4-way handshake = instant recovery (~10s).

Script for macOS:
```bash
ip=$(ipconfig getifaddr en0)
case "$ip" in 169.254.*) networksetup -setairportpower en0 off; sleep 2; networksetup -setairportpower en0 on ;; esac
```

### 🟠 Lower leverage
- Static IPs on iPhone/MBP (bypasses DHCP entirely)
- Separate SSID for "stable" vs "lab" clients (different BSSID = different GTK)
- Different AP for different client groups

### 🔴 Avoid the trigger entirely
- **Don't plug wired client into same SW1 fabric while same client's Wi-Fi is associated**
- Disable Wi-Fi BEFORE plugging Ethernet
- Use separate switch for the wired path

## Recommended implementation order for the lab
| # | Action | Effort | Reduction |
|---|--------|--------|-----------|
| 1 | DHCP lease 7200s → 86400s on SW1 | 1 cmd | Significant |
| 2 | Enable 802.11r in XIQ on Corporate SSID | XIQ toggle | Re-assoc <100ms |
| 3 | GTK Update Interval 1h → 5 min | XIQ toggle | Bounds miss window |
| 4 | Recovery script watching APIPA | Medium | Sub-10s |
| 5 | Procedural: Wi-Fi off → plug en8 → Wi-Fi on | None | Eliminates trigger |

## Open questions for GTAC / R&D
1. Is this a known issue in current AP3000 firmware?
2. Expected behavior of Group Key Handshake retry when M2 ACK is missed from lagging client?
3. Are the configurable knobs the right mitigation, or is there a firmware fix?

## Key data for engineering escalation
- Platform: AP3000 + EXOS 5320, XIQ-managed
- Encryption: WPA3-SAE-PMF, AES-CCMP, ch 161 5GHz, 11ax
- Trigger: deterministic, dual-interface event
- Symptom: 2-of-N clients fail, trigger device immune, 2-4 min self-recovery
- Reproduced multiple times, consistent behavior
- Tech-support dumps + 24MB pcap available in repo data folder
- AP EAPOL trace NOT captured (kdebug wsec flooded SSH; recommend XIQ Device 360 → Tools → SSH/CLI for future kdebug to use cloud-relay path)

## Linked resources
- Verbatim dialogue (engineering escalation reference): `docs/data/may8-dhcp-repro/dialogue_pcap_analysis_mitigations_May8_2026.md` (online: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/may8-dhcp-repro/dialogue_pcap_analysis_mitigations_May8_2026.md)
- Incident write-up: `project_dhcp_macbook_incident.md`
- Triage runbook: `reference_dhcp_wifi_triage_runbook.md`
- Pcap: `docs/data/may8-dhcp-repro/pcap_May8_workmbp_en0_repro.pcapng`
- Full session DOCX: `docs/data/may8-dhcp-repro/ClaudeAI_SessionDetails_May8_1209PST.docx`