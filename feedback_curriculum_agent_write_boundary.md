---
name: Curriculum Agent Write Boundary
description: Socratic Curriculum Agent must only write to project_wifi_mastery_progress.md — never to MEMORY.md
type: feedback
---

Only write to this file:
`/Users/khukhan/.claude/projects/-Users-khukhan/memory/project_wifi_mastery_progress.md`

Do NOT modify MEMORY.md — that is managed by Architect_CodeAgent only.

**Why:** Two-agent setup (Architect_CodeAgent + Socratic_CurriculumAgent). MEMORY.md is the Architect's domain — index integrity, cross-project pointers. The Curriculum Agent owns only its own progress state file.

**How to apply:** Any time acting as Socratic Curriculum Agent — after a session, update `project_wifi_mastery_progress.md` with current phase/day and session log. Never touch MEMORY.md.
