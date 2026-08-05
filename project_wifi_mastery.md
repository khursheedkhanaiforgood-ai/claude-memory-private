---
name: WiFi Mastery Repo — Site Structure
description: wifi-mastery GitHub Pages site. As of Apr 29 2026: NYT landing page is root, dark knowledge base moved to docs/. Full nav hierarchy established.
type: project
originSessionId: f514f622-f559-4492-9f78-451fab154098
---
## Repo
- GitHub: https://github.com/khursheedkhanaiforgood-ai/wifi-mastery
- GitHub Pages root: https://khursheedkhanaiforgood-ai.github.io/wifi-mastery/

## Site Hierarchy (as of Apr 29 2026)

```
index.html                          ← NYT landing page (FRONT DOOR)
docs/wifi-mastery-guide.html        ← dark knowledge base (was root index.html)
docs/hld-wifi-digital-twin.html     ← HLD v1.1
docs/lld-phase1-wifi-digital-twin.html ← LLD Phase 1 v1.0
docs/design-journey.html            ← 7-theme ADR (Architecture Decision Record)
docs/session_summary_20260429.html  ← EOD Apr 29
docs/index-nyt.html                 ← OLD copy (duplicate of root, leave as-is)
```

**Why:** NYT page is the hub linking to everything. Dark site is content; NYT page is navigation.

## NYT Landing Page (root index.html)
- Sticky top nav: ● WiFi Mastery | Knowledge Base | Architecture Docs | Design Strategy | Sprint Roadmap | Design Journey | Session Log | HLD | LLD | EOD
- Sections: Start Here → Knowledge Base (22 sections, all clickable) → Architecture Docs → Design Strategy → Sprint Roadmap → 5G↔WiFi Bridge → AP Fleet → Design Journey card → Session Logs
- Design Journey is a card linking to docs/design-journey.html (NOT inline)

## Dark Knowledge Base (docs/wifi-mastery-guide.html)
- Dark-themed, sidebar nav, 22 sections inline (JS show/hide)
- Hash navigation supported: docs/wifi-mastery-guide.html#rf works
- Sidebar has ⚡ Digital Twin Platform → ../index.html
- Home pillar grid has 5th card → Digital Twin Platform

## Knowledge Base Content (22 sections)
- 01-design/: D1 RF Fundamentals, D2 SSID Architecture, D3 Channel Planning, D4 Security, D5 QoS, D6 VLAN/I-SID
- 02-configuration/: C1 XIQ Policy, C2 SSID Profiles, C3 User Profiles, C4 Guest Isolation, C5 Fabric Attach
- 03-deployment/: P1 AP Placement, P2 PoE, P3 XIQ Onboarding, P4 Deployment Checklist
- 04-troubleshooting/: T1 Client Connectivity, T2 Roaming, T3 RF Interference, T4 Debug Commands, T5 XIQ Diagnostics
- 05-reference/: R1 802.11 Standards, R2 AP3000 Notes, R3 Glossary

## Next session
- Sprint 1 Day 1: write src/ Python code (models/dpm.py, engines/link_budget.py, engines/airtime.py)
