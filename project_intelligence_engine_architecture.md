---
name: XIQ→EP1 Intelligence Engine — Architecture Decisions
description: Full architectural decisions, design rationale, and strategic choices from May 21 2026 design session. Reference for all future sprints.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---

## What the System Is

A GPS navigator for network UI configuration — not a lookup table, not a document search. Per step: exact portal, exact left-nav item, exact tab, exact field label, exact dropdown value, exact button to click. Version-pair aware.

**Analogy locked:** "GPS navigator for UI configuration. XIQ is your starting map. EP1 is your destination. The engine knows every road, one-way street, and detour — including roads that EP1 hasn't built yet (gaps)."

---

## Core Design Decisions — Locked May 21, 2026

### Query Classifier (locked decision)
Every user query enters a classifier FIRST before any graph traversal:
- **workflow** → "show me the workflow for AP VLAN tagging" → full multi-step sequence, Neo4j traversal
- **lookup** → "which menu to configure CWP for PPSK?" → single Neo4j node, nav path only
- **compare** → "what's different between XIQ and EP1 for switch policy?" → side-by-side diff table

Both query types return same confidence scoring. Lookup short-circuits — no full traversal.

### 3-Layer Data Architecture (locked)
L1 = RAG (14 PDFs → pgvector), c_doc = 0.3
L2 = GUI Capture (Playwright → Neo4j), c_gui = 0.4
L3 = Live Lab (Execution Agent → PostgreSQL), c_lab = 0.3

Confidence formula (ported from Cisco EN CLI Agent): Σ(w×c)/Σ(w)
When c_lab absent: renormalise over c_doc+c_gui only (0.3/0.7 + 0.4/0.7)

Confidence tiers: HIGH ≥0.70 · MED 0.50–0.69 · LOW 0.40–0.49 · GAP <0.40

### RAG vs Scrape — 4 Outcomes (locked)
1. RAG says X at path A + Scraper finds X at path A → **confirmed** (high confidence)
2. RAG says X at path A + Scraper finds X at path B → **restructured** (medium confidence, new nav path)
3. RAG says X exists + Scraper finds nothing → **gap** (logged to Gaps Register)
4. Scraper finds X + RAG silent → **ep1_only** (new EP1 capability, document it)

### 17-Agent System (locked, do NOT simplify to fewer)
12 LangGraph nodes (0=Orchestrator through 11=Feedback Loop) + 5 supporting agents.
User pushed back when I tried to simplify to 5 nodes — full 12 are required.
Supporting agents: Playwright Scraper, PDF Ingestor, Gap Curator, Version Manager, Schema Migrator.

### Why React, Neo4j, Neo4j were kept (user corrections)
- React: NOT replaced by plain HTML/JS — React needed for version picker + state management
- Neo4j: NOT replaced by pgvector-only — Neo4j is for WORKFLOW GRAPHS (traversal), pgvector is for SEMANTIC SEARCH over docs. Different data structures, different queries.
- 11→12 nodes: user explicitly pushed back on simplification

### Version-Pair Architecture (locked)
All mappings tagged to exact version pair. XIQ-25.8.0→EP1-25.9.0 stored separately from XIQ-25.8.0→EP1-25.9.2. Users pick pair from top-nav dropdown. Admins trigger re-scrapes on new EP1 patch.

### Confidence Formula Source
Ported directly from Cisco EN CLI Agent (cisco-en-cli-agent repo). Same weights, same Σ(w×c)/Σ(w) formula. DO NOT change weights without testing against known-good step pairs.

---

## RAG Corpus (14 PDFs)

Location: /Users/khukhan/Downloads/EP1 and Other Items for Migration/
Priority order for Sprint C ingestion:
1. Feature Comparison PDF — chunk_weight=2.0 (explicit XIQ→EP1 mapping, ground truth)
2. EP1 User Guide 25.9.0
3. EN_Hub_XIQ-EP1.pdf
4. Security User Guide 25.6.0
5. Universal Switch Deployment Guide 25.7.0
6. Release Notes (patches 1–4) + FAQ + transition white papers + licensing guides

---

## Repo Structure

New repo: /Users/khukhan/xiq-ep1-intelligence-engine/ (local, Sprint A committed f0c85f8)
GitHub repo: NOT YET CREATED — must run `gh repo create xiq-ep1-intelligence-engine --public`
Design doc: xiq-ep1-intelligence-engine.html on feature/xiq-ep1-intelligence-engine branch of 5320-onboarding
GitHub Pages: NOT LIVE until merged to main

---

## Sprint A Status (COMPLETE — May 21, 2026)
- 57 files, commit f0c85f8
- 25 unit tests: 25/25 passing
- Node 05 (confidence formula): IMPLEMENTED
- Node 06 (gap detector): IMPLEMENTED
- All other nodes: typed stubs with NotImplementedError
- Seeds: 31 XIQ→EP1 bins + G-01/G-02/G-03

---

## Strategic Decision — Two-Page Architecture

**xiq-ep1-transition.html** = Living practitioner guide / research artifact / gap register
- Keep as-is. Do NOT restructure it.
- Contains: step-by-step sprint guide, gap register (G-01/G-02/G-03+), feature mapping research, the "why"
- When engine is deployed: add final section "The Engine — Built From This Research" + link to deployed app

**xiq-ep1-intelligence-engine.html** = Design + architecture doc
- Grows as sprints complete
- Contains: system design, agentic architecture, sprint plan, automated validation

**Cross-linking rule (set May 21):**
- From design doc: reference specific #section anchors of transition guide for rationale
- From engine (deployed): link back to transition guide gap register showing which gaps informed which feature
- Transition guide is the research artifact; engine is the deliverable
- Relationship: transition guide → motivates → design doc → specifies → engine → links back

---

## Hard Deadline

**July 1, 2026: XIQ GUI retired.** Sprint B (Playwright XIQ scraper) is the irreversible bottleneck.
Every XIQ screen must be captured and version-tagged in Neo4j BEFORE this date.
No workaround exists after July 1. Sprint B is the #1 priority after Sprint A infrastructure is stable.

---

## Next Actions (as of May 21 EOD)
1. Merge feature/xiq-ep1-intelligence-engine to main → make design doc live on GitHub Pages
2. `gh repo create xiq-ep1-intelligence-engine --public` → push Sprint A code
3. Sprint B: Playwright XIQ scraper — start IMMEDIATELY, run before July 1
4. AP1 network policy: Employee PPSK VLAN5_Employee + Guest CWP VLAN10_Guest (lab deploy, TBD)
5. Fix Port 3 in EP1 switch template (set Admin Enabled) to prevent G-03 on re-push
