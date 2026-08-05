---
name: XIQ→EP1 Architecture Design Dialogue — May 15 2026 (ongoing)
description: Verbatim design conversation for the XIQ→EP1 Navigation Intelligence Engine. Two-agent RAG scrape results, thematic bins, front-end architecture, context engine hypothesis, multi-agent AI architecture, 5-day sprint plan, DB schema, graph DB decision.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Design Dialogue — May 15 2026 (verbatim)

### User's Brain Map (exact)
> (1) Go through ALL buttons, understand what each does and why
> (2) Deploy EXOS and AP policies as I have done to navigate
> (3) Deploy same on EP1 steps (1) and (2)
> (4) Scour the RAG-database of ALL the EN documents published to date on XIQ and on the new EP1 — user guide, release notes, solutions_architecture documents, deployment guides. Compile all in a linked-list bibliography. Two agents doing these two tasks in parallel.
> Front-end: ALL CURRENT XIQ GUI items mapped to EP1 GUI. Template = Cisco→EXOS CLI mapping agent with 'thematic bins of data/functions' independent of XIQ or EP1. Then figuring out from the XIQ_database and the EP1_database — whether they match, how they match, the heat map etc.
> Context engine: when the user asks a question, deploy an AP1 policy in EP1 — I know how to do it in XIQ but need to do it in EP1.

### Hypothesis (defined May 15)
Build a **XIQ→EP1 Navigation Intelligence Engine** modelled on the Cisco→EXOS CLI agent.
- Instead of CLI translation: **GUI path translation**
- Thematic bins of functions **independent of either portal**
- Each bin: XIQ nav path + EP1 nav path + match type + context steps
- Context engine answers: "I know how to do X in XIQ — show me EP1"
- Heat map shows coverage across all bins

### Architecture Layers
1. **Two corpora** — XIQ docs + EP1 docs (user guides, release notes, solution arch, deployment guides)
2. **Thematic bins** — 25 functional areas (confirmed from two-agent RAG scrape)
3. **Mapping layer** — per bin: XIQ path ↔ EP1 path + gap register
4. **Context engine** — "deploy KB_School AP1 policy" → surfaces XIQ steps + EP1 equivalent inline
5. **Heat map** — visual coverage: green=matched, amber=restructured, red=missing/EP1-only
6. **Front-end** — web UI, dialogue-based design; template = Cisco→EXOS CLI agent

### Two-Agent RAG Scrape Results (May 15)

#### Agent 1 — XIQ Documentation
- Primary portal: https://supportdocs.extremenetworks.com/support/documentation/extremecloud-iq/
- Developer portal: https://developer.extremecloudiq.com/documentation/
- Latest user guide: v25.9.0 Classic (online) + v25.2.0 (PDF)
- 30 functional areas confirmed across 9 document categories
- Key docs: User Guide v25.2.0, Release Notes v25.9.1, Switch Deployment Guide, Universal Switch Guide
- Feature-specific guides: PPSK (GUID-E4BFDC21), SSID config, firewall, Client 360, LDAP, AirDefense, RF/Smart RF
- GitHub: extremenetworks/ExtremeCloudIQ-APIs (400+ Postman requests), OpenAPI spec

#### Agent 2 — EP1 Documentation
- Primary portal: supportdocs.extremenetworks.com/support/documentation/ep1-latest-documentation/
- EP1 has TWO modules: Networking (maps to XIQ) + Security/ZTNA (new, separate user guide)
- Latest: EP1 Networking v25.9.0, EP1 Security v25.6.0
- UI Transition FAQ (community PDF): ExtremeCloud IQ UI Transition - CustomerFAQ.pdf
- Universal Switch Deployment Guide: EP1 v25.7.0
- Licensing Guide: EP1 and XIQ 25.6.0 (combined)
- NO official XIQ→EP1 migration guide found
- New-in-EP1: AI Canvas (NL dashboard), ZTNA, TPME (third-party mgmt), MSP multi-tenancy

### Critical Timeline Correction (from EP1 agent)
- July 1, 2026 = XIQ New UI becomes MANDATORY DEFAULT (not full retirement)
- Q4 2026 = XIQ Classic UI fully retired
- Sprint target stays July 1 (clients must be on new UI)
- New UI has been available since July 2025

### 25 Thematic Bins (confirmed from both corpora)
| # | Bin | Match |
|---|-----|-------|
| 1 | Device Onboarding | Both — nav changed |
| 2 | Device Templates (AP/Switch/Router) | Both |
| 3 | Network Policies | Both — renamed/restructured |
| 4 | SSID / Wireless Networks | Both |
| 5 | Switch Policies + Port Types | Both |
| 6 | VLAN Management | Both |
| 7 | RF / Radio Profiles | Both |
| 8 | PPSK + User Groups | Both |
| 9 | 802.1X + RADIUS | Both |
| 10 | Captive Web Portal / Guest | Both |
| 11 | Firewall + Traffic Filters | Both |
| 12 | User Profiles + VLAN Assignment | Both |
| 13 | Monitoring / Dashboards | Both + EP1 AI Canvas |
| 14 | Client 360 + Analytics | Both |
| 15 | Alarms + Events | Both |
| 16 | Reports | Both |
| 17 | Rogue AP / AirDefense | Both |
| 18 | Location / Planning / Topology | Both |
| 19 | Admin Accounts + RBAC | Both |
| 20 | API + Integrations | Both (same API) |
| 21 | Licensing | Both — structure changed |
| 22 | Firmware Updates | Both |
| 23 | Audit Logs | Both |
| 24 | Backup + Restore | Both |
| 25 | ZTNA + Third-Party (TPME) | EP1-only |

### XIQ vs EP1 Key Structural Differences
| Dimension | XIQ | EP1 |
|-----------|-----|-----|
| Module split | One product | Networking + Security (ZTNA) — separate |
| AI | Basic analytics | AI Canvas (NL query), Agent Manager, autonomous agents |
| Third-party | Extreme devices only | TPME — manages any vendor |
| MSP | Not optimized | Purpose-built multi-tenant, consumption billing |
| Licensing | XIQ Pilot/Advanced/Premium | EP1 Standard (=XIQ Pilot + EP1) / Advanced / Premium |
| Official migration | None | No guide — UI Transition FAQ only |

### Front-End Design Direction (to develop next session)
- Template: Cisco→EXOS CLI agent (thematic bins, confidence/match scoring, side-by-side view)
- Input: user selects a bin OR types a task ("configure PPSK")
- Output: XIQ path (left) ↔ EP1 path (right) + match badge + steps + doc links
- Heat map: 5×5 grid of all 25 bins, colored by match status
- Bibliography: linked list of all docs from both agents (live links)
- Context engine query: "I want to [task in XIQ terms]" → returns EP1 equivalent workflow

### Key Docs to Seed the Front-End
- XIQ User Guide v25.2.0 PDF
- EP1 Networking User Guide v25.9.0
- EP1 Security User Guide v25.6.0
- XIQ UI Transition FAQ (community PDF)
- EP1 Licensing Guide v25.6.0
- EP1 Universal Switch Deployment Guide v25.7.0

### How to Apply
- Every week's session log feeds findings back into the mapping layer
- Thematic bins = Week 1-4 sprint topics (4 bins per week ~= 4-6 bins/week)
- Front-end HTML file: xiq-ep1-engine.html (to be built in dialogue session)
- Live-link this memory in the EOD HTML for XIQ→EP1 sprint

---

## Design Dialogue — May 15 2026 Session 2 (verbatim)

### User Decision: Independent Architecture

> "I don't want to share the same database as the CISCO -- EN CLI. I want to build an independent database for the XIQ -- EP1 transition. I do want to minimize effort. But I want to keep these Railway deployments independent of each other. Is that possible? This XIQ -- EP1 will be a monumental effort and there will be MANY EYES on it, so the architecture, motivation, front-end, back-end, production-ready deployment for other teams to test as well... and contribute to, the RAG... all will be under the microscope."

**Decision reached:** Fully independent repo, independent Railway project, independent PostgreSQL + pgvector instance. ZERO shared infrastructure with cisco-en-cli-agent. Reuse patterns (confidence calc, bin router, fallback agent logic) but not the runtime.

### User Requirements (exact)
> "I want to build a multi-agent architecture where the lead agent choreographs from intake (from users) to delivery. While sub-agents do the leg-work. Clearly define the AI_Architecture. I was told a graph-database or something similar maybe useful, 'perhaps in addition to RAG' not sure? Also, I want to have an agent like we had in the CISCO -- EN CLI one where, if there is no match, no path from XIQ -- EP1 then it goes out to the internet and crawls for new data or asks the user for inputs etc. Do you have that architecture mapped here from the CISCO -- EN CLI one? ALSO, and this is very important, I want ALL of our dialogue as we are building the architecture, debating on choices and finally deciding on a path ahead to be linked list to this dialogue verbATIM."

### Architecture Decisions Made (Session 2)

#### Option B Confirmed
- FastAPI backend (Railway) + static HTML/JS frontend (GitHub Pages)
- Independent Railway project, independent PostgreSQL
- Plain HTML/JS NOT React (contributor-friendly, no build pipeline, works with GitHub Pages)

#### Graph Database Decision
**Problem:** XIQ→EP1 is not a lookup table — it is a navigation graph. Features have dependency chains (e.g., cannot configure 802.1X without RADIUS; cannot configure SSID without Network Policy existing first). Graph traversal answers "what is the full dependency chain to accomplish X in EP1" — vector search alone cannot.

**Decision:**
- Week 1: PostgreSQL adjacency tables (simulate graph, no new service)
- Week 2+: Migrate to Neo4j AuraDB free tier when traversal queries get complex
- LangGraph = agent orchestration graph (NOT data storage — different thing)
- pgvector = semantic search over doc corpus (retained)

Three data layers in one Railway Postgres instance:
1. pgvector — semantic search ("what feature is the user asking about?")
2. Adjacency tables (→ Neo4j) — graph traversal (dependency chains, relationship edges)
3. Standard tables — bins, mappings, sessions, confidence scores, bibliography

#### Fallback Agent (3-Tier) — Ported from Cisco→EXOS
Exact same escalation pattern, new search targets:
- Tier 1: Bin-scoped DB search + graph traversal
- Tier 2: Full semantic search (pgvector, all 25 bins)
- Tier 3: Web crawl → supportdocs.extremenetworks.com → community.extremenetworks.com → developer.extremecloudiq.com → prompt user → write back to DB

#### Multi-Agent Architecture (LangGraph StateGraph)
```
Lead Orchestrator (LangGraph StateGraph)
  ├── Intake Agent (parse intent, detect bin, route)
  ├── XIQ Navigator Agent (lookup XIQ nav path in corpus + graph)
  ├── EP1 Navigator Agent (lookup EP1 equiv path in corpus + graph)
  ├── Confidence Agent (Σ(wᵢ×cᵢ)/Σ(wᵢ), IDENTICAL/RESTRUCTURED/PARTIAL/GAP/EP1_ONLY)
  ├── Context Engine Agent ("I know XIQ flow X → full EP1 workflow")
  ├── Gap Detector (confidence < 0.40 → route to Fallback)
  ├── Fallback Agent (3-tier search + web crawl + user prompt loop)
  ├── Response Synthesizer (XIQ path | EP1 path | badge | doc links | dependency chain)
  ├── Heat Map Agent (async — updates 25-bin coverage grid)
  └── Bibliography Agent (async — maintains linked-list of all sources)
```

#### DB Schema (graph-compatible from Day 1)
```sql
features(id, product, bin_tag, feature_name, nav_path, doc_url, embedding vector(1536))
feature_edges(id, from_id, to_id, edge_type)
  -- edge_type: maps_to | requires | replaces | gap | ep1_only
mappings(id, xiq_id, ep1_id, match_type, confidence, bin_tag, week_verified, notes, verified_by)
bibliography(id, product, doc_title, doc_url, version, doc_type, ingested_at)
```

### 6-Day Sprint A Plan (6hr/day) — REVISED with Browser Agents
| Day | Bucket | Key Deliverables |
|-----|--------|-----------------|
| 1 | Foundation | Repo + Railway + DB schema + FastAPI skeleton + LangGraph skeleton |
| 2 | RAG + Corpus | Ingestion pipeline + 31-bin seed + mapping seed + confidence calc ported |
| 3 | Agent Build | Orchestrator + Intake + Navigator + Confidence + Gap Detector wired E2E |
| 4 | Fallback + API | Fallback agent + Bibliography + Heat map + all FastAPI endpoints |
| 5 | Browser Agents | Playwright setup + XIQ Browser Agent + EP1 Browser Agent + Cross-Validation Agent + GitHub Actions cron |
| 6 | Frontend + Deploy | HTML/JS wired to API + GitHub Pages + Railway deploy + E2E test + CONTRIBUTING.md |

### Dual-Stream Validation + 75% Phase Gate (Session 6)

> "I want both, the documentation based validation/heat map as well as the 'actual one run on the XIQ/EP1' platforms. The reason is, I don't know which one will ultimately work because NO_ONE has tested all of the EP1 features and done anything like this previously. So, let's run those in parallel until the DB is '75%' or so full by experiment/confidence on actual XIQ/EP1. We then break out of 'the debug/optimization' guardrail and move into fully the validated RAG/Graph space?"

**Decision:** Both streams run in parallel until db_coverage ≥ 0.75.

Phase gate logic:
- `db_coverage = COUNT(verified_by='browser_agent') / COUNT(all mappings)`
- Below 0.75 → PARALLEL mode: every query triggers both streams; divergences feed back into RAG corpus
- At/above 0.75 → RAG_VALIDATED mode: doc stream primary, browser agents spot-check only

Self-calibration loop: wrong doc entry → correction → corpus update → RAG re-indexes → future predictions improve.

Heat map is now DUAL: doc confidence (left) vs verified confidence (right) per bin.
- Both green → fully validated bin
- Doc green, verified amber → partially live-tested
- Doc green, verified red → docs wrong → active investigation

New agent added: **Validation Orchestrator** — monitors coverage %, enforces phase gate transition.

### Browser Automation Validation Agents (Session 5)
Three Playwright-based agents added to architecture:
- **XIQ Browser Agent** — login XIQ → navigate specified path → screenshot → verify feature exists → return verified nav_path
- **EP1 Browser Agent** — login EP1 (9-dot launcher) → navigate → screenshot → verify → return
- **Cross-Validation Agent** — compares both outputs, updates DB: `verified_by='browser_agent'`, confidence=1.0 if match, GAP flag if missing

Trigger modes: on-demand (per query), weekly GitHub Actions cron, on-push PR validation gate.
Credentials: XIQ + EP1 same login (extremecloudiq.com). Stored in Railway env vars. Handle Day 5.
Impact: confidence shifts from doc-based to empirically validated. Heat map becomes audit trail.

### React Decision
> "I don't understand [React] well enough."
**Decision: Plain HTML/JS.** Reasons: no build pipeline, GitHub Pages compatible, contributors are network engineers not frontend developers, fetch() calls FastAPI in three lines. React deferred — would add tooling friction for no UI benefit at current complexity level.

### File Residency Decision
- `xiq-ep1-transition.html` → stays in 5320-onboarding forever (sprint observer / personal learning log)
- `xiq-ep1-design-dialogue.html` → lives in 5320-onboarding NOW; moves to `xiq-ep1-intelligence-engine/docs/` on Day 1 of Sprint A when new repo is created
- 5320 xiq-ep1-transition.html then links out to new GitHub Pages URL for the engine

### Repo Structure (confirmed)
```
xiq-ep1-intelligence-engine/
├── backend/          ← FastAPI on Railway
│   ├── api/          ← REST endpoints (/search /bins /heatmap /confidence /session)
│   ├── rag/          ← ingestion pipeline, vector search
│   └── agents/       ← LangGraph orchestrator + all sub-agents + confidence.py
├── frontend/         ← Static HTML/JS → GitHub Pages
├── corpus/
│   ├── xiq/          ← XIQ docs
│   └── ep1/          ← EP1 docs
├── database/
│   └── migrations/   ← SQL files, versioned, contributable
├── docs/             ← architecture.md, CONTRIBUTING.md, RAG guide
└── tests/            ← pytest suite
```

---

## Design Dialogue — May 15 2026 Session 3 (verbatim)

### User Additions (exact)
> "I want two sprints, one with plain HTML and one with React. Also, I need to learn about graph, engineering/architecture and implementation. Finally, these xiq-ep1 will reside independently on GitHub, but will be accessible from the 5320 landing page. I want you to consider the usage, user is new to EP1. He/she is building a policy to deploy KB_School. It involves both AP and EXOS config. After policy is created he/she will push. But then, the user will need to know where to go on EP1 to observe performance, 'tune or calibrate'. In essence, the E2E workflow for observability, configuration, automation, orchestration... the whole nine yards. Do you have these elements in the 25 bins? What are we missing?"

### Decisions Made (Session 3)

#### Sprint Plan: Two Sprints
- **Sprint A (5 days, plain HTML)** — full product: backend + agents + HTML/JS frontend + deploy
- **Sprint B (3 days, React — frontend only)** — Sprint A backend stays, frontend migrated to React components; pedagogical sprint to learn React

#### 25 Bins → 31 Bins (E2E gap analysis)
E2E scenario traced: new user builds KB_School AP + EXOS policy, pushes, observes, tunes, automates.

**Missing bins identified:**
| # | Bin | Why Added |
|---|-----|----------|
| 26 | Policy Push / Config Deploy | Delta vs full push, push history, rollback, reboot-after-push — discrete critical workflow not in any existing bin |
| 27 | EP1 AI Canvas + NL Query | Paradigm shift, no XIQ equivalent. NL dashboard, anomaly detection. Was buried in #13 |
| 28 | EP1 Agent Manager + Autonomous Agents | EP1-only. Rule-based or NL-triggered autonomous agents. No XIQ equivalent |
| 29 | Smart RF + Radio Optimization | Was buried in #7. Full workflow: channel, power, coverage hole detection |
| 30 | Troubleshooting + Diagnostics | Completely absent. Packet capture, connectivity test, tech support dump, live client diagnostics |
| 31 | MSP + Multi-Tenancy | Separate from ZTNA (#25). Consultant multi-org management, tenant switching, consumption billing |

#### Graph Education Decision
Three tools, distinct purposes:
- **LangGraph** = agent workflow graph (control flow, not data storage)
- **pgvector** = semantic search over doc corpus (fuzzy text matching)
- **Neo4j / graph DB** = stores knowledge as nodes + edges (data storage, path-finding, dependency chains)

Implementation path:
- Week 1: PostgreSQL adjacency tables (graph-shaped schema, no new service)
- Week 2+: Migrate to Neo4j AuraDB free tier (200K nodes / 400K edges, native Cypher queries)

Schema is designed graph-compatible from Day 1 — migration = data export + import, not schema redesign.

#### GitHub + 5320 Integration
- Repo: `khursheedkhanaiforgood-ai/xiq-ep1-intelligence-engine` (independent)
- Frontend: GitHub Pages at `khursheedkhanaiforgood-ai.github.io/xiq-ep1-intelligence-engine/`
- Backend: own Railway project (own PostgreSQL)
- 5320 landing page banner: TWO links → "Launch Engine" (app) + "Sprint Log" (xiq-ep1-transition.html)

#### React Explained (for user's background)
React components = XIQ templates. A Network Policy template defines a pattern applied to many devices. A React component defines a UI pattern rendered many times with different data (props). In plain HTML: copy 31 bin cards. In React: write 1 BinCard component, pass 31 data objects. Sprint B teaches this by migration.
