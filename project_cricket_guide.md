---
name: cricket_guide_project
description: State of the Cricket Guide app — CLI + Streamlit GUI, deployed to GitHub and Streamlit Cloud
type: project
---

## Cricket Guide — Where We Left Off

Two apps built. CLI works fully. Streamlit GUI deployed but something not working (to be debugged next session).

### Apps on disk
- `~/cricket-guide/` — CLI version (fully working)
- `~/cricket-guide-gui/` — Streamlit GUI version (deployed, needs debugging)

### GitHub repos
- CLI tutorial output: https://github.com/khursheedkhanaiforgood-ai/cricket-guide
- GUI source code: https://github.com/khursheedkhanaiforgood-ai/cricket-guide-gui

### Published URLs
- Static tutorial (GitHub Pages): https://khursheedkhanaiforgood-ai.github.io/cricket-guide/
- Live interactive app (Streamlit Cloud): https://share.streamlit.io → cricket-guide-gui app

### GitHub username
`khursheedkhanaiforgood-ai` (note: .env had a bug where it was set to `khursheedkhanaiforgood.ai` with a dot — was fixed)

### Tech stack
- Agent 1 (Research): `claude-sonnet-4-6` + Tavily search (async, httpx)
- Agent 2 (Publisher): `claude-haiku-4-5` + GitHub REST API (requests)
- GUI: Streamlit (`app.py`) — uses `concurrent.futures.ThreadPoolExecutor` to bridge async agents into Streamlit's sync context
- Search: Tavily only (Google was removed at user's request)

### Required API keys (stored in .env, also added to Streamlit Cloud secrets)
- `ANTHROPIC_API_KEY`
- `TAVILY_API_KEY`
- `GITHUB_TOKEN`
- `GITHUB_USERNAME=khursheedkhanaiforgood-ai`

### Next session: debug Streamlit Cloud deployment
- User deployed to Streamlit Cloud and added secrets via Settings → Secrets tab
- Something not working — need to check: app logs on share.streamlit.io, secrets format (must be TOML), and whether the app is actually rebooting after secrets are saved

**Why:** App runs fine locally but Streamlit Cloud deployment has an unresolved issue.
**How to apply:** On next session, start by asking user what error they see, then check Streamlit Cloud logs.
