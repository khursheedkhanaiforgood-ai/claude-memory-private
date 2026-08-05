---
name: XIQ → EP1 Transition Sprint
description: 6-week sprint (May 15 – Jul 1 2026) to exhaustively document every XIQ feature mapped to EP1, for client guidance. XIQ GUI retired Jul 1.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Goal
XIQ GUI is being retired July 1, 2026. Khursheed has 6 weeks to document every XIQ feature/workflow, find the EP1 equivalent, identify gaps, and produce a client-ready transition guide.

**Why:** Clients will be forced onto EP1 on July 1 with no fallback. Khursheed needs to be the expert who guides them through the navigation and workflow changes.

## Timeline
| Phase | Dates | Focus |
|-------|-------|-------|
| Week 1 | May 15–21 | Device management (onboarding, firmware, templates, IQAgent) |
| Week 2 | May 22–28 | Network policies (switch, wireless, VLAN, Supplemental CLI) |
| Week 3 | May 29–Jun 4 | User access & security (PPSK, 802.1X, CWP, guest) |
| Week 4 | Jun 5–11 | Monitoring, alerts, reports, admin, API |
| Week 5 | Jun 12–18 | Independent testing — reproduce all workflows in EP1 without notes |
| Week 6 | Jun 19–25 | Client-ready documentation, printable quick-reference card |
| Buffer | Jun 26–Jul 1 | Final review, publish before deadline |

## Branch & Files
- Branch: `feature/xiq-ep1-transition` on 5320-onboarding repo
- Main page: `xiq-ep1-transition.html` (NYT style, purple #7c3aed accent)
- Linked from all 3 index pages (index.html, index-nyt.html, index-harpers.html)
- Each week's session logs link back from the main transition page

## EP1 Key Facts (from reference_ep1_extreme_platform_one.md)
- EP1 = Extreme Platform One, accessed via 9-dot launcher in current XIQ portal
- NOT the IPE SD-WAN device
- When EP1 is down: deploys blocked, PPSK RadSec fails — check cloud status page
- Underlying API is unchanged between XIQ and EP1

## How to Apply
- Every session during the 6-week sprint should produce an EOD HTML log
- Feature coverage map in xiq-ep1-transition.html tracks status (Done/In Progress/Pending/Gap)
- Gaps register accumulates any features missing or changed significantly in EP1
- Week 5 is INDEPENDENT — no notes allowed, pure muscle memory test
- Final deliverable: client quick-reference card (Week 6)
