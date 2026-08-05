---
name: June 30 2026 — RADIUS Migration, Tagged/Untagged VLAN, Remote Passive Capture
description: AP2 moved to Karl's VIQ for RADIUS testing; CAPWAP debug journey; tagged vs untagged deep-dive; Sacramento remote passive capture strategy
type: project
originSessionId: 4d3a8b8b-a7df-4354-a376-1ed6b280263d
---
AP2 (AH-565780) migrated to Karl's VIQ (VHM-SXJUBHJO, server 3.145.235.74) for RADIUS 802.1X testing. CAPWAP reached RUN state. RADIUS policy pushed via EP1.

**Why:** RADIUS auth test against Karl's NPS server — user credentials not yet in Karl's RADIUS DB, deferred to post-PTO.

**How to apply:** When resuming RADIUS sprint, Karl adds user creds first, then re-run `_debug auth basic` + `debug console` on AP2 console.

## CAPWAP redirect lessons (June 30)
- XIQ Classic and EP1 are separate systems — redirector only follows EP1 claims (extremeplatformone.com)
- Must delete from EP1 inventory (not just Devices list) or AP stays mapped to old org
- Default AP password: `aerohive` (older IQ Engine firmware)
- After EP1 policy push, password changes — use EP1 device admin policy credential
- CAPWAP states: SULKING → DTLS Handshaking → RUN

## Tagged vs Untagged (confirmed mental model)
- Wireless clients NEVER tag frames — AP tags on their behalf
- Untagged/access port = all SSIDs collapse to VLAN 1, no network isolation possible
- Tagged/trunk port = AP maps each SSID to a VLAN ID, switch routes accordingly
- Inter-station blocking = only within same VAP/SSID, not cross-SSID
- EP1 firewall = L3 only; can't block intra-VLAN L2 traffic
- Karl's no-tagging client requirement = accept single VLAN, no isolation between SSIDs

## Sacramento remote passive capture (Phase 1)
Old AP (2019, no SSH, lost password, no policy history) on HPE switch, user in Seattle.
Three remote options:
1. `ipconfig /all` from switch port (laptop replaces AP) — 80% info, APIPA = trunk smoking gun
2. Screen share + Wireshark walkthrough — 95%
3. Client captures pcapng + emails — 100%, you analyze locally

What to request from client:
- `netsh wlan show networks mode=bssid` screenshot (or Option+WiFi on macOS)
- `ipconfig /all` screenshot from switch port
- Wireshark pcapng (30s, no filter, from switch port)

Phase 2 (factory reset + EP1 claim) may require on-site if no console access.

## EOD
Local: `/Users/khukhan/5320-onboarding-agent/docs/session_summary_20260630_radius_vlan.html`
Live: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/docs/session_summary_20260630_radius_vlan.html
