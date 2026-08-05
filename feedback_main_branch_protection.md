---
name: eve-ng-lab-platform main branch protection
description: The main branch of eve-ng-lab-platform is locked — no changes without explicit user password authorization
type: feedback
---

The `main` branch of the `eve-ng-lab-platform` repo (owner: khursheedkhanaiforgood-ai) is protected by the user. No changes may be pushed to main unless the user explicitly authorizes it by providing "the year of my older daughter's birth" as a password/authorization signal.

**Why:** User wants to preserve the working stable version and avoid accidental breakage. Quote: "This version should from now on be completely protected E2E. No changes allowed unless I share the password of the year of my older daughter's birth."

**How to apply:** Before making ANY commit or push to `main` on this repo, stop and ask the user for authorization. New features go to separate branches (e.g. `feature/traceroute`). The user will merge to main when ready.
