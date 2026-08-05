---
name: GitHub token — cisco-en-cli-agent
description: GitHub PAT location for khursheedkhanaiforgood-ai/cisco-en-cli-agent — token is in project .env, remote URL already includes it
type: reference
---

GitHub repo: https://github.com/khursheedkhanaiforgood-ai/cisco-en-cli-agent
Owner: khursheedkhanaiforgood-ai

PAT storage:
- Saved in: /Users/khukhan/Projects/cisco-en-cli-agent/.env as GITHUB_TOKEN=...
- Also embedded in git remote URL (so `git push` works without prompting)
- NOT in memory (security) — read from .env when needed

To push from any future session:
  cd /Users/khukhan/Projects/cisco-en-cli-agent
  git push origin main
  (token is in the remote URL — no prompt)

To read token for API calls:
  source /Users/khukhan/Projects/cisco-en-cli-agent/.env  (or grep GITHUB_TOKEN from it)
