---
name: EXOS Layer 2 Ring Lab — Shareable HTML (EVE-NG)
description: 6-switch EXOS ring lab guide + STP troubleshooting session HTML. Current state, live URLs, pending work.
type: project
---

## What Was Built
Shareable 3-tab HTML document covering an EXOS Layer 2 ring lab built in EVE-NG:
- **Tab 1** — Full lab guide: Steps 1–8, annotated CLI commands, improved SVG topology diagram
- **Tab 2** — VPCS/STP troubleshooting: root cause analysis with actual CLI outputs
- **Tab 3** — Loop Behaviour: why adjacent switch pings worked but VPCS cross-ring pings failed

## Live URLs
- **Public:** https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/exos-layer2-lab.html
- **Private session log (#me):** https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/exos-layer2-lab.html#me
- **GitHub file:** khursheedkhanaiforgood-ai/5320-onboarding → main branch → `exos-layer2-lab.html`
- **Local copy:** /Users/khukhan/Downloads/EXOS-Layer2-Lab-Guide-Shareable.html

## #me Hidden Section
- Yellow banner (large font, black bg, yellow borders) fixed at top when URL has #me fragment
- "View session log" button opens a dark modal overlay
- Modal contains the FULL 9-step sequential troubleshooting session log with actual CLI outputs:
  Step 1: Symptom reported → Step 2: Port/VLAN verified OK → Step 3: ARP asymmetry (loop signature)
  → Step 4: FDB wrong port (Client1 MAC on trunk) → Step 5: show stpd detail = Ports:(none) root cause
  → Step 6: EXOS vs Cisco PVST+ explanation → Step 7: Fix on all 6 switches
  → Step 8: Core-4 port 2 → BLK → Step 9: Ping restored ✓

## Pending (for next session — March 31 2026+)
- **Tab 4** (user requested): Move the session log OUT of the hidden #me modal and into a proper visible 4th tab
  User said: "Use a fourth tab and don't have to hide it"
  → Create Tab 4 "Session Log" with the same 9-step content, sidebar nav, visible to everyone
  → #me banner can remain but Tab 4 makes it fully public/accessible

## Lab Details
- 6-switch ring: Core-1, Core-2, Core-4, Core-5 (backbone) + Edge-1, Edge-2 (access)
- VLANs: Data (10), Voice (20), Kh_Mgmt (30) — trunked on ports 1 & 2 on all switches
- VPCS clients: 2 per VLAN, one on each edge switch
- Root cause: EXOS STPD s0 enabled but Participating VLANs: (none) on all 6 switches
- Fix: configure stpd s0 add vlan Data/Voice/Kh_Mgmt ports 1,2 on all 6 switches
- Result: Core-4 port 2 → BLK, pings restored

## GitHub Push Method
No `gh` CLI available. Use Python urllib + git credential-osxkeychain:
```python
token = subprocess.check_output('printf "protocol=https\nhost=github.com\n" | git credential-osxkeychain get', shell=True)
# Then GET SHA, PUT with base64 content to GitHub Contents API
```
