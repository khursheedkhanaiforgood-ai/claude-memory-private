---
name: Deep Research Engine — Phase 0 complete, architecture review pending May 26
description: deep-research-engine repo at /Users/khukhan/deep-research-engine — Phase 0 scaffold done (8/8 tests, commit 4cd6b90); user wants architecture decision review on 2026-05-26 before Phase 1 starts
type: project
originSessionId: 00942a66-67ef-4dab-a50a-79497ab744d5
---
Phase 0 scaffold complete 2026-05-25. Commit: `4cd6b90` on `main`.

**Project path**: `/Users/khukhan/deep-research-engine/`
**Package name**: `dre` — `from dre.llm.router import complete`

**IMPORTANT — May 26 action item**: User explicitly wants to **revisit the architecture decision** before starting Phase 1. Do NOT jump into Phase 1 code. Open PLAN.md, walk through the key architectural choices, and confirm with user before coding.

Key decisions to review:
- LangGraph 1.0 (StateGraph supervisor) vs alternatives
- LiteLLM unified layer vs direct provider SDKs
- STORM outline-first discipline — user needs to confirm this adds value for their use case
- Tri-Agent independence enforcement via `independence_firewall.py` — confirm structural pattern
- DiscoUQ (arXiv 2603.20975) vs simpler consensus mechanisms
- Model assignments: GPT-5.5 as Checker, Gemini-3.1-pro as Auditor — confirm API access

**Architecture (approved in PLAN.md, but subject to May 26 review)**:
- LangGraph 1.0 (StateGraph, supervisor, checkpointing, HITL interrupt_before)
- LiteLLM unified API layer (all 3 vendors via one interface)
- Three independent model roles: Generator (Claude) / Consistency Checker (GPT-5.5) / Transparency Auditor (Gemini-3.1-pro)
- STORM outline-first + GPT-Researcher breadth=4/depth=2 recursive tree
- AutoGen-style per-claim Critic gate (Phase 4)
- Tri-Agent Audit with `independence_firewall.py` chokepoint (Phase 5)
- SelfCheckGPT + DeBERTa-v3 NLI → TU = AU + EU (Phase 6)
- DiscoUQ structured disagreement (NOT majority voting)

**Phase 0 — files created (all complete)**:
- `pyproject.toml` — uv/hatch, all deps locked
- `src/dre/config.py` — frozen Config, model IDs, cost guardrail ($5/run)
- `src/dre/llm/router.py` — LiteLLM wrapper, sole legal LLM path, `--probe` CLI
- `src/dre/state/schema.py` — ResearchState TypedDict (14 keys)
- `tests/conftest.py` + `tests/fixtures/mock_llm_responses.json`
- `tests/test_phase0_scaffold.py` — 8 smoke tests, all passing
- `CLAUDE.md`, `BACKLOG.md`, `.claude/rules/` (3 rules), `.claude/settings.json`
- `.env.example`, `.gitignore`

**One remaining Phase 0 item**:
- User must fill `.env` with 3 API keys → run `python -m dre.llm.router --probe`
- Confirms live connectivity to Anthropic + OpenAI + Google before Phase 1 starts

**Phase sequence (10 total)**:
0→Scaffold✓ | 1→Single-LLM baseline | 2→Multi-perspective (STORM) | 3→Research tree | 4→Critic loop | 5→Tri-Agent Audit | 6→Hallucination | 7→Output schema | 8→Stopping + HITL | 9→Production

**Open engineering questions** (surface during architecture review):
1. DeBERTa-v3-mnli — CPU only or MPS (Apple Silicon)?
2. Gemini Grounding API metadata — does LiteLLM expose it?
3. GPT-5.5 API access confirmed? Fallback: `openai/gpt-4.5` or `openai/o3`
4. LangGraph checkpointer — MemorySaver (Phase 1) or SQLite (Phase 8+)?
5. DiscoUQ (arXiv 2603.20975) — reference impl available?
6. SelfCheckGPT — pip `selfcheckgpt` or reimplement from paper?
7. CRAG web fallback — Tavily or SerpAPI?

**Why:** User wants multi-agent deep research with hard cross-LLM corroboration. No single model trusted. Self-hostable, not SaaS.

**How to apply — NEXT SESSION START SCRIPT:**
1. Say: "Welcome back. Last session (May 28) we finished Phase 0 — 8/8 tests, committed. One thing blocked us from Phase 1: the architecture review you flagged. Want to do that now?"
2. If yes → open PLAN.md, walk through the decisions table one by one, get explicit confirmation on each
3. Only after confirmed → Phase 1 files: `src/dre/agents/researcher.py`, `src/dre/agents/synthesizer.py`, `src/dre/graph.py`
4. Remind user to fill `.env` + run `python -m dre.llm.router --probe` before any live LLM calls
