---
name: VOSS Digital Twin Simulator — Full Architecture & Sprint State (April 8 2026)
description: FabricEngine Flight Simulator — SOA Python + Streamlit Sprint 1 built, Railway deployment PENDING. Sprint 2=React+persistence+pgvector. Architecture decisions, 3-plane view, export engine.
type: project
---

## What Was Built

**FabricEngine Flight Simulator** — professional-grade simulation for EXOS→VOSS migration.
Analogy: flight simulator for pilots / NASA astronaut training — practice everything before touching real hardware.

**Repo:** https://github.com/khursheedkhanaiforgood-ai/voss-fabric-migration
**Local:** /Users/khukhan/voss-fabric-migration
**Run:** `streamlit run streamlit_app.py`
**Sprint 1 Status:** BUILT but NOT COMPLETE — Railway deployment required to close Sprint 1.

## Architecture Decisions (jointly agreed April 8 2026)

| Decision | Sprint 1 | Sprint 2 |
|----------|----------|----------|
| Session persistence | In-memory (st.session_state) | PostgreSQL or Redis |
| Authentication | admin / Extreme01!! | XIQ login or LDAP |
| AI explanations | On-demand (student types `explain`) | Always-on streaming |
| UI framework | Streamlit | React |
| Deployment | Railway (Nixpacks) | Railway (Docker) |

## The "Flight Simulator" Concept

The simulator is NOT a quiz tool. It is:
- A professional test environment like NASA training or 737 flight sim
- Every command maps to real hardware behavior
- Outputs (CLI scripts, XIQ JSON) are DEPLOYABLE to physical switches
- The "cockpit" shows all three network planes simultaneously

## Welcome Page Structure (as of April 8 EOD)

1. Hero header (NYT-style)
2. "Start the Simulation" auth block (admin/Extreme01!!)
3. Theory sections (currently inline — NEXT SESSION: wrap in st.expander dropdowns):
   - "What Is This Migration and Why Does It Matter?"
   - "How does a phone get IP through the fabric?" (dark table)
   - "Why IS-IS? What is SPBM?"
   - "What is XIQ's Role?" (dark purple card — added April 8 afternoon)
   - "The 18 Steps as One Sentence Each" (7-theme story rows)
4. Learning Path linked list
5. Three-Plane Architecture
6. Lab Topology (ASCII st.code block)
7. Migration Themes (7 colored bins)
8. Cisco-EN → VOSS Mapping (expander)
9. Full Standards Reference (expander)

**PENDING next session:**
- Wrap ALL theory sections in st.expander() dropdowns (collapsed by default)
- Move auth block to top (right after hero) so users can skip theory
- Verify HTML rendering fix is live (unsafe_allow_html=True already present)

## Three-Plane Cockpit View (the key UX innovation)

Three rows, time-aligned per migration step:

**Management Plane** (blue, #326891)
- XIQ: intent, policy push, device templates, ZTP+
- iqagent status, network policy actions

**Control Plane** (green, #1a7a3a)
- IS-IS (RFC 6329): adjacency, system-id, manual-area
- SPBM (IEEE 802.1aq): ethertype, nick-name
- Fabric Attach (IEEE 802.1Qcj): LLDP-FA TLVs, FA assignment state

**Data Plane** (purple, #6B21A8)
- MAC-in-MAC (IEEE 802.1ah): fabric encapsulation
- I-SID services: E-LAN active/not-active
- Traffic path: phone → AP → switch → NNI → modem → internet

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

## Branches

- `main` — production simulator (no socratic mode)
- `feature/socratic-mode` — full Socratic implementation (hide answers, dialogue log, user-script export, diff view). Preserved for Sprint 2 consideration.

## Key Files

| File | Purpose |
|------|---------|
| streamlit_app.py | Main Streamlit app — all pages (welcome, simulator, export) |
| app/export_engine.py | CLI script + XIQ JSON + checklist generator |
| simulator/config.py | Lab constants + STANDARDS_EXOS + STANDARDS_FABRIC (with URLs) |
| simulator/models/migration_step.py | 18 steps with theme, why, exos_parallel, standard_url |
| simulator/services/explain_service.py | ExplainService — Claude API with step context as RAG |
| simulator/services/state_machine_service.py | 18-step FSM with jump_to_step() |
| railway.json | Railway deploy config |
| docs/session_summary_20260408.html | Full EOD session report |

## Railway Deployment (Sprint 1 blocker)

railway.json and Procfile already committed. Steps to deploy:
1. `railway login` (browser auth)
2. `railway init` (link to project)
3. `railway up`
4. Set env var: ANTHROPIC_API_KEY in Railway dashboard
5. Set env var: STREAMLIT_SERVER_PORT=$PORT (handled in railway.json)
URL pattern: https://voss-fabric-migration-production.up.railway.app

## Sprint 2 Plan (NOT YET BUILT)

- React frontend (3-panel: step tracker | terminal | lab dashboard)
- FastAPI backend with WebSocket
- 5 discrete agents: OnboardingAgent, FabricConfigAgent, VerificationAgent, OptimizationAgent, SessionLogAgent
- AgentOrchestrator with handoff protocol
- pgvector RAG corpus (VOSS CLI PDF + IEEE standard summaries)
- Persistent sessions (Redis or PostgreSQL)
- XIQ auth (replace admin/Extreme01!!)
- Socratic mode from feature/socratic-mode branch (reviewed + integrated)
