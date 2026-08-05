---
name: Test before committing to Git
description: User expects code to be verified before pushing — don't commit guesses that will fail in CI/Railway
type: feedback
---

Test or verify before committing, especially for deployment configs (Railway, Docker, Nixpacks). Pushing untested config changes that fail in Railway wastes deploy cycles and erodes trust.

**Why:** During the Railway deployment session (March 25 2026), five consecutive failed builds resulted from guessing at the Nixpacks environment without local verification. User explicitly called this out: "Are you testing them before you commit to Git?"

**How to apply:** Before pushing deployment config changes, either: (1) test locally with Docker/nixpacks CLI if available, or (2) explicitly tell the user that local testing is not possible and explain why, rather than silently pushing guesses. When Docker/nixpacks is not installed, say so upfront and propose the most conservative approach first.
