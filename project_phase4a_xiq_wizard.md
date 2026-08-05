---
name: Phase 4a — XIQ Guest Onboarding Wizard
description: Intent-based XIQ config wizard (Phase 4a, two-week sprint, manager approval needed). NLP parsing layer + deterministic XIQ API generator + RAG backend. Confirmed May 12.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Goal
Deploy the PPSK Guest Onboarding Wizard — user types natural-language intent → wizard asks clarifying questions → generates XIQ objects via API.

## Phase Roadmap
| Phase | Timeline | Description |
|-------|----------|-------------|
| 1 HTML | Done May 12 | ppsk_simulator.html — 3 scenes, 24 steps |
| 2 Node.js | May 13 | Express wrapper + REST API layer + Docker (simple simulator wrapper — SEPARATE from Phase 4a) |
| 3 Railway | May 13 | Deploy to Railway → public URL |
| 4 NVIDIA/SW-Defined | 2 weeks — **manager approval needed** | NVIDIA platform deployment of Digital Twin |
| 4a XIQ Wizard | Same sprint as 4 | Intent → NLP → XIQ API → deploy |

## Phase 4a Architecture (confirmed May 12)
- XIQ flow IS deterministic (100% match once intent is fully understood)
- Confidence calculator lives at the NLP/parsing layer ONLY — for decoding ambiguous English intent, NOT for XIQ config generation
- Analogous to CISCO EN CLI Mapping Agent (Sprint 14) — same confidence/parsing pattern, different output (XIQ API calls vs EXOS CLI)

## RAG Backend Plan
- XIQ REST API docs (all SSID/UG/UP/Assignment Rule endpoints)
- PPSK Guide v6 (Rieben, 95pp) + RFC annotations
- EXOS 33.x + VOSS FE 9.3.x CLI references
- RFCs: 6614, 2898, 5415, 2865, 2868, 2131, 802.11-2020, 802.11i, 802.1Q, 802.1X
- ExtremeGuest Essentials docs (for Phase 2/3 CWP option)

## Railway = Multi-Agent Framework (confirmed May 12)
- User decided Phase 4a Railway backend = multi-agent framework, NOT single-service Express
- Agents needed: NLP/parsing agent, XIQ API agent, RAG query agent, confidence scoring agent
- **Socratic session planned: 2026-05-13 morning** — derive agent boundaries from first principles
- Phase 2/3 Node.js (simple HTML simulator wrapper) is SEPARATE from the multi-agent XIQ wizard

## Why:** Manager approval needed for Phase 4 (NVIDIA platform cost + resourcing). Phase 4a adds RAG ingestion sprint that needs scheduling. Two-week minimum window.
## How to apply:** Do NOT plan Railway backend as single Express app. Multi-agent is the architecture. Socratic May 13 morning defines the agent boundaries before any coding.
