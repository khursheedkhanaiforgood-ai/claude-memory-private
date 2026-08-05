---
name: Multi-Session Write Safety
description: Four concurrent Claude Code sessions run simultaneously — never overwrite shared files without reading current state first
type: feedback
---

There are up to 4 concurrent Claude Code sessions running at any time. All sessions share:
- `/Users/khukhan/.claude/CLAUDE.md`
- `/Users/khukhan/.claude/projects/-Users-khukhan/memory/MEMORY.md`
- `/Users/khukhan/.claude/projects/-Users-khukhan/memory/project_wifi_mastery_progress.md`

**Rule:** Always Read a shared file immediately before editing it — never edit from a cached version. Another session may have written to it since you last read it.

**Why:** Concurrent edits without reading first silently overwrite another session's work.

**How to apply:**
- Read → Edit (never Edit from context alone) on any shared memory file
- When appending to MEMORY.md Active Alerts, read the full section first
- When updating project_wifi_mastery_progress.md, read it first even if you wrote to it earlier in the same session
