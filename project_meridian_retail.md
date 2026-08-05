---
name: Meridian Retail Group — Separate Engagement
description: Meridian is a separate customer from Horizon. Consultant exercise Apr 23 2026. Do NOT conflate with Horizon Karl Lab work.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
Meridian Retail Group is a **separate customer engagement** from Horizon Custom Fabrication.

**Why:** User explicitly flagged on 2026-04-28 that Meridian must not be mixed with Horizon. They are separate projects.

**How to apply:** When referencing Apr 23 EOD content, clearly label Horizon items (SW2 recovery, Karl team review) separately from Meridian items (consultant exercise). Never group them as one engagement.

## Meridian Retail Group — Apr 23 2026 Consultant Exercise

Vendor consultant review of IT Manager D. Kowalski's WiFi 7 upgrade proposal (6 retail stores + Atlanta warehouse, AP4020s + 5320 stacks). Goal: practice consultant-grade pushback as Extreme vendor.

**Block 1 — Discovery Phase**
- Customer: "Ready to order equipment and start Phase 1 in May."
- Pushback: Need field site survey, legacy inventory, traffic throughput data, physical layouts, specific issue documentation before any order.

**Block 2 — Standardization Trap**
- Customer: "One network policy for all 7 sites."
- Pushback: Warehouse (85,000 sq ft, handheld scanners, 200ft range, 2.4 GHz critical) ≠ retail store (confined, high-density POS, 5/6 GHz). EP1 enables site-specific policies — not blanket config.

**Block 3 — Security Architecture**
- Customer: "POS + Guest share same subnet. App-layer encryption is sufficient."
- Pushback: App-layer encryption doesn't prevent ARP spoofing, DoS, metadata snooping, switch-room compromise. Network and application security are separate layers.

Source: EOD_Blueprint_Apr23_2026.html → section id="meridian"
