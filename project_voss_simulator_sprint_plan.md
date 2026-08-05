---
name: VOSS Simulator — Sprint Plan & Pending Work (April 8 2026 EOD)
description: Sprint 1 BUILT (Railway deploy pending = blocker). Pending next session: theory expanders, copyright footer, Railway deploy.
type: project
---

## Sprint 1 Status — BUILT, NOT COMPLETE

**Blocker:** Railway deployment. Do first in next session.
**Steps:** `cd /Users/khukhan/voss-fabric-migration && railway login && railway up`
Then set ANTHROPIC_API_KEY in Railway dashboard.

## Welcome Page — Current State (as of April 8 EOD)

Structure (committed to main, working on port 8502):
1. Hero header (NYT-style)
2. **Auth block** — moved to top so users can skip theory ✅
3. `---`
4. Theory sections (inline, NOT yet in expanders — deferred to next session):
   - "What Is This Migration and Why Does It Matter?" (blue card)
   - "How does a phone get IP through the fabric?" (dark table, 6 steps)
   - "Why IS-IS? What is SPBM?" (green card)
   - "What is XIQ's Role?" (dark purple card)
   - "The 18 Steps as One Sentence Each" (7 colored rows)
5. Learning Path (linked list)
6. Three-Plane Architecture (3 columns)
7. Lab Topology (ASCII st.code block)
8. Migration Themes (7 colored bins)
9. Migration Mapping expander (already collapsed) ✅
10. Full Standards Reference expander (already collapsed) ✅

## Pending Next Session (in order)

1. **Railway deployment** — Sprint 1 blocker
2. **Theory → st.expander() dropdowns** — wrap each of the 5 theory sections
   in `with st.expander("...", expanded=False):` and indent all content inside.
   NOTE: requires indenting all content (4→8 spaces) — do as one complete rewrite
   of the theory block, not piecemeal edits.
3. **Copyright + footer** — user asked for copyright and other footer items on
   the welcome page. Needs: © 2026 Extreme Networks / Lab Series, lab version,
   links to GitHub repo, session report. Match the dark-themed EOD HTML footer style.
4. **GitHub Pages** — enable on voss-fabric-migration repo so EOD HTML is public.
   Settings → Pages → main branch → root. URL: khursheedkhanaiforgood-ai.github.io/voss-fabric-migration/

## 7 Functional Themes (Bins)

| Theme | Steps | Description |
|-------|-------|-------------|
| OS-CONVERT | 1–3 | Backup, OS change, ZTP re-adoption |
| ISIS-CONTROL | 4–6 | Router ISIS, SPBM instance, enable |
| NNI-LINK | 7 | NNI port, IS-IS on port |
| VLAN-ISID | 8–9 | VLAN creation + I-SID E-LAN bindings |
| ANYCAST-DHCP | 10–11 | Anycast gateways + DHCP server (SW1) |
| ACCESS-FA | 12–14 | Fabric Attach, Internet exit, IP shortcuts |
| SAVE-VERIFY | 15–18 | Save config + all verification steps |

## Sprint 2 — Planned (NOT building yet)

1. PostgreSQL + pgvector corpus
2. FastAPI backend + WebSocket
3. React frontend (3-panel)
4. 5 agents: OnboardingAgent, FabricConfigAgent, VerificationAgent, OptimizationAgent, SessionLogAgent
5. Persistent sessions
6. XIQ auth
7. Review + integrate feature/socratic-mode branch

## ExplainService (Sprint 1)

File: simulator/services/explain_service.py
Trigger: student types "explain", "why", "why [concept]"
Fallback: built-in step.why if no ANTHROPIC_API_KEY
Sprint 2: replace _build_context() with pgvector cosine search

## Key Runtime Info

- Local: port 8502 (started with --server.port 8502)
- Credentials: admin / Extreme01!!
- Git branches: main (production), feature/socratic-mode (frozen, Sprint 2)
