---
name: Multi-LLM Research Framework Plan
description: Full approved plan for greenfield multi-LLM deep research framework — 8 phases, 4-layer hallucination detection, Claude/GPT-4o/Gemini fan-out, awaiting implementation start
type: project
originSessionId: 31fb3c19-b9fc-44f5-a589-ed75ca7301af
---
Plan approved 2026-05-24. Implementation NOT yet started — user paused to check other processes.

**Project path**: `/Users/khukhan/multi-llm-research/` (greenfield, does not exist yet)

**Goal**: Fan a research query to Claude (claude-sonnet-4-6 workers, claude-opus-4-7 arbiter), GPT-4o, and Gemini 1.5 Pro in parallel. Cross-validate claims through a 4-layer hallucination stack. Return `ConsensusReport` JSON with per-claim verdicts.

**Why:** User wants NotebookLLM-style deep research but multi-model, with explainable hallucination detection. First corpus candidate = WLPC 150+ WiFi files (already in memory as reference_wlpc_corpus.md).

**Tech stack (finalised)**:
- Python 3.12, direct provider SDKs (NOT LangChain), LangGraph orchestration
- FastAPI + arq/Redis backend, Streamlit UI (NOT React)
- ChromaDB embedded vector store (abstract interface, swappable)
- Pydantic v2, structlog, ruff, pytest-asyncio

**4-Layer Hallucination Detection**:
- L1 Cross-LLM: embed-cluster claims across providers — sole-provider = flag
- L2 Source grounding: every claim must cite a chunk_id; missing/low-cosine = flag
- L3 Self-consistency: paraphrase + re-query same provider + compare (toggleable)
- L4 Arbitration: Claude Opus 4.7 meta-judge → CONFIRMED/DISPUTED/HALLUCINATION_LIKELY/UNVERIFIABLE

**8 Phases**:
| # | Phase | Size |
|---|-------|------|
| 0 | Scaffold, config, pyproject | S |
| 1 | Ingestion → chunker → embedder → ChromaDB | M |
| 2 | Async fan-out Claude/GPT-4o/Gemini + normalizers | L |
| 3 | 4-layer hallucination engine + scorer | XL |
| 4 | Consensus synthesizer → ConsensusReport | S |
| 5 | FastAPI REST + WebSocket + arq jobs | M |
| 6 | Streamlit UI (Ingest/Research/Verdict Inspector) | S |
| 7 | LangGraph wiring (8 nodes, fast/thorough modes) | M |

**Critical path**: 0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 (Phases 1+2 parallelisable after 0)

**Key risks**: cost blowup (up to 9 LLM calls/query), Gemini JSON compliance, claim alignment across providers, arbiter hallucinating its own verdict.

**Mirroring**: pyproject.toml + config.py pattern from `/Users/khukhan/xiq-ep1-intelligence-engine/`

**How to apply**: When user says "start the framework" or "begin Phase 0" — start at scaffold step, do NOT re-plan.
