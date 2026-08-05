---
name: Memory Cleanup — Always Ask Before Removing or Trimming
description: Never remove, trim, or archive memory entries without explicit user approval. Propose the changes and wait for confirmation.
type: feedback
originSessionId: 3db2d693-1e37-4eb5-8f10-fc421a5db74d
---
Always ask before removing, trimming, or archiving any memory entry.

**Why:** User explicitly requested this after entries were removed without permission on 2026-08-05. No data should be silently deleted — it all represents real work context.

**How to apply:**
- When MEMORY.md is over the size limit: present a specific list of proposed removals/trims and ask for approval before touching anything.
- When creating an archive: show the user what will move and where, confirm before proceeding.
- "Obvious" duplicates or clearly stale entries still need approval — what looks stale may still be needed.
- Exception: entries the user explicitly says to delete in the current conversation.
