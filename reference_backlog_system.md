---
name: BACKLOG.md session-start system
description: Every project should have a BACKLOG.md at root + a CLAUDE.md session-start rule; any Claude session reads it first and briefs the user
type: reference
originSessionId: 00942a66-67ef-4dab-a50a-79497ab744d5
---
## The pattern

Each project repo gets two things:

1. **`BACKLOG.md`** at the project root — a living to-do list with three sections:
   - **Immediate** — unblocked, do now
   - **Deferred** — parked items with context
   - **Completed** — recent wins (keep last ~10)

2. **`CLAUDE.md` session-start rule** — first section, above everything else:
   > Read `BACKLOG.md` and give the user a 3–5 bullet brief of what's pending before doing anything else.

## First instance

`/Users/khukhan/5320-onboarding-agent/BACKLOG.md` — created 2026-05-24.
Sprint: Index Parity + EOD Cleanup (May 22 card missing from all 3 index pages; May 11 missing from 2 pages).

## Maintenance rules (tell Claude in any session)

- "update the backlog" → add new pending items mid-session
- "what's pending?" → brief without full re-read
- Mark `[x]` as work completes, not at end of session
- Parity rule: every EOD HTML in `docs/` must get a card on all 3 index pages (index.html, index-nyt.html, index-harpers.html)

## Apply to new projects

When initialising any new project repo, create `BACKLOG.md` + add the session-start rule to `CLAUDE.md` as the first section.
