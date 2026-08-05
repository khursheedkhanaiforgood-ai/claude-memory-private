---
name: Session Startup Briefing — All 4 Sessions
description: On every login, read MEMORY.md Active Alerts and brief the user on the state of all concurrent sessions before doing anything else
type: feedback
---

On every session start, before taking any action:

1. Read `/Users/khukhan/.claude/projects/-Users-khukhan/memory/MEMORY.md` — specifically the `## 🔔 Active Alerts` section
2. Brief the user on ALL active sessions in a short bullet list — what each is working on, what's blocked, what's ready to start
3. Ask which thread they want to pick up

**Why:** Up to 4 concurrent Claude Code sessions run simultaneously across different projects. The user needs a single consolidated briefing on login rather than having to mentally track each session themselves.

**Format for the briefing:**
> Here's where all sessions stand:
> - **[Session name]** — [status] · [next action]
> - ...
> Which would you like to continue?

**How to apply:** Every session start, regardless of working directory. Don't wait for the user to ask.
