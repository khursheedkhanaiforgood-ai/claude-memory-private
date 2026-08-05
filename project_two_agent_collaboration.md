---
name: Two-Agent Collaboration Setup
description: Architect_CodeAgent (Claude Code) + Socratic_CurriculumAgent dual-agent workflow for 802.11 curriculum and multi-LLM research framework. Shared state via memory file.
type: project
originSessionId: 31fb3c19-b9fc-44f5-a589-ed75ca7301af
---
Established 2026-05-24.

## Agent Roster
| Agent | Name | Role |
|-------|------|------|
| This session (Claude Code) | **Architect_CodeAgent** | Framework building, deliverable generation, gap analysis, curriculum gate design, corpus cross-referencing |
| Second Claude window | **Socratic_CurriculumAgent** | Daily Socratic sessions, pacing, questioning, session log writing |

## Shared State File
`/Users/khukhan/.claude/projects/-Users-khukhan/memory/project_wifi_mastery_progress.md`
Both agents read and write this file. Architect_CodeAgent writes Handoff Notes. Socratic_CurriculumAgent writes Session Logs and updates Current State.

## Two Active Threads
1. **802.11 Mastery Curriculum** — 6-phase Socratic curriculum (34 days). Socratic_CurriculumAgent runs sessions; Architect_CodeAgent reviews logs, generates deliverables, writes gate questions.
2. **Multi-LLM Research Framework** — Greenfield Python project at /Users/khukhan/multi-llm-research/. Plan approved, implementation NOT started. Architect_CodeAgent builds this solo.

## Daily Routine
1. Open Socratic_CurriculumAgent window → it reads shared file → runs session → writes log
2. Switch to Architect_CodeAgent window → say "check curriculum state" → gap analysis + deliverable generation + handoff notes
3. When ready: say "start Phase 0" to begin multi-LLM framework build

## Kickoff Prompt for Socratic_CurriculumAgent
Paste this to start it:
"You are my Socratic Curriculum Agent for the 802.11 Mastery Curriculum. Before each session read: /Users/khukhan/.claude/projects/-Users-khukhan/memory/project_wifi_mastery_progress.md — check Current State for phase/day, check Handoff Notes for flags from Architect_CodeAgent. Run today's Socratic session, then append the session log block and update Current State. Let's begin — read the file and tell me where we are."

**How to apply**: When user says "check curriculum state" — read project_wifi_mastery_progress.md first, then respond. When user says "start Phase 0" — begin multi-LLM framework build from project_multi_llm_research_framework.md plan.
