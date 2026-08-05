---
name: Concurrent Claude Code Sessions
description: Up to 4 Claude Code sessions run simultaneously — never overwrite shared files without re-reading first; minimize writes to MEMORY.md and CLAUDE.md
type: feedback
originSessionId: 31fb3c19-b9fc-44f5-a589-ed75ca7301af
---
User runs up to 4 concurrent Claude Code sessions at any time.

**Why:** Multiple specialist agents are active in parallel (e.g. Architect_CodeAgent, Socratic_CurriculumAgent, and others). Each may write to shared memory files.

**How to apply:**
1. Always `Read` a shared file immediately before `Edit` — never assume it hasn't changed since you last read it
2. Make the smallest possible edit — one targeted `Edit` block, not a full rewrite
3. Prefer appending to the end of MEMORY.md over inserting in the middle (lower conflict risk)
4. For session-specific state, write to a dedicated per-session file rather than a shared one
5. If an edit fails with "file modified since read" — re-read, reconcile, then write (do not blindly retry the same edit)
6. MEMORY.md and CLAUDE.md are the highest-conflict files — treat every write to them as a critical section
