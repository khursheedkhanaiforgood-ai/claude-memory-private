---
name: PPSK Self-Registration Gap + 802.1X Future Sprint
description: EP1 v25.9.0 has no domain filter for PPSK self-registration. Proper solution is 802.1X + RADIUS. Remind user when NPS/RADIUS is available.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Finding — May 20, 2026

EP1 v25.9.0 has **no domain allowlist** for PPSK self-registration. There is no UI field to restrict CWP registration to @extremenetworks.com or any other domain. Confirmed by full search of User Guide pages 265-315, Table 105 (Cloud User Group), Table 113 (CWP features).

**Why:** Product gap — not documented as planned either. Domain filtering likely requires API config or future release.

## What Was Learned — The Toggle

"Return Aerohive Private PSK" toggle exists in CWP SSID features (Table 113, p.295):
- Open SSID + CWP + Enable Self-Registration + Return Aerohive Private PSK
- Employee fills form → EP1 auto-generates PPSK from Cloud User Group → delivers via email/SMS
- Employee disconnects → connects to secure PPSK SSID → VLAN 5
- Two-SSID architecture required (open onboarding SSID + secure PPSK SSID)

## Future Sprint — 802.1X + RADIUS/NPS

**How to apply:** When user mentions NPS, RADIUS, Active Directory, or 802.1X — remind them this is the parked sprint from May 20. The 802.1X PEAP-MSCHAPv2 arc was first designed May 11 (dot1x_simulator.html). Pick up from there.

**Solution design:**
- Replace PPSK on KhKLab_Employee with 802.1X
- RADIUS/NPS validates corporate credentials against AD
- Domain membership = automatic enforcement (no UI domain filter needed)
- VLAN 5 returned as RADIUS attribute on success
- Sprint guide: ge-airdefense-xiq-sprint.html#ppsk-selfreg
