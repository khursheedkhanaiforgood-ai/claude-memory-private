---
name: EP1 = Extreme Platform One (not IPE SD-WAN)
description: EP1 refers to Extreme Platform One, the next-gen XIQ cloud portal accessed via the 9-dot launcher in XIQ. NOT the IPE SD-WAN device in Path B.
type: reference
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---

**EP1 = Extreme Platform One** — Extreme Networks' next-generation cloud management portal, successor to the classic XIQ UI. Accessed via the 9-dot app launcher inside XIQ.

When EP1 is "down":
- XIQ web portal (management GUI) is unreachable
- Policy deploys are blocked
- AP still runs the last-pushed policy locally (Corp/Guest SSIDs keep working)
- PPSK RadSec calls may be affected (per-auth cloud dependency)
- Corp/Guest may show "no internet" if AP fails-closed on User Profile enforcement

EP1 being down ≠ internet down. Check extremenetworks.com/support/cloud-services-status for outage status.

**Do NOT confuse with IPE** — the IPE is the SD-WAN device in Path B (SW2 → IPE Port 1/1 → RDU/FishBowl DC GRE tunnel). Completely separate device and function.
