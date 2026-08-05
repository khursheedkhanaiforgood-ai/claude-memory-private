---
name: Engine Sprint E — Two UI Query Modes
description: Sprint E UI design decision: one-shot fast mode (no HITL, confidence score shown) + guided two-step HITL mode. Deferred from May 21 session.
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Two Query Modes (deferred to Sprint E)

### Mode 1 — One-Shot / Fast Mode (no HITL)
- For users in a hurry
- Single query → immediate response with confidence score displayed
- Confidence factor shown for: (a) language/intent parse quality, (b) overall workflow confidence
- No route preview gate — engine executes and returns
- Button label TBD (e.g. "Quick Answer" or "Fast Mode")

### Mode 2 — Guided / HITL Mode (two-step)
- User sees route preview before any execution (Node 0 Phase 1 output)
- Shows per-step confidence, gap warnings, warn-level steps
- User confirms or redirects before engine executes
- Option for user to guide the agentic flow mid-traversal (Node 6 pause)
- Button label TBD (e.g. "Guide Me" or "Step-by-Step")

## Why
User: "keep it for a separate sprint, the one shot without HITL where a confidence factor is given for the language/intent and the flow, for when someone is in a hurry. But there is a button too, for the two step HITL as well or to have the user guide the agentic flow."

## How to Apply
When designing Sprint E (React frontend): default to one-shot fast mode as the primary UX. HITL guided mode is a toggle/button. Both modes use the same backend graph — only the HITL gate and route_preview rendering differ. Do NOT build both modes simultaneously — ship fast mode first, add guided mode in Sprint E.2 or F.
