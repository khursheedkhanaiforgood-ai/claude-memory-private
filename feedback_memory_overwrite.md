---
name: Never overwrite existing memory files without user confirmation
description: User said "Do NOT overwrite anything" when Claude overwrote project_multi_llm_research_framework.md — always restore + create new file, never replace silently
type: feedback
originSessionId: 00942a66-67ef-4dab-a50a-79497ab744d5
---
Never silently overwrite an existing memory file, even if it appears stale or factually wrong.

**Why:** On 2026-05-25, Claude overwrote `project_multi_llm_research_framework.md` (an earlier draft plan) with current state. User caught it mid-write and said "Wait, what memory are you changing? Do NOT overwrite anything." The original had historical value even though the architecture had changed.

**How to apply:** When a memory file needs updating:
1. Show the user what the old file says vs. what you want to write
2. Offer three options: (a) keep update, (b) restore original + new file, (c) review both and decide
3. Only proceed after user confirms
4. When in doubt, always prefer creating a new file alongside the old one — never replace
