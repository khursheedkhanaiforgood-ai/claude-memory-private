---
name: 5320-onboarding repo — GitHub Pages serves from main
description: Critical config fact for the 5320-onboarding GitHub Pages site. Pages serves from main branch, NOT gh-pages. EODs must land on main to go live. Three landing pages must stay in parity.
type: reference
originSessionId: 3370a62e-c170-4c60-bde0-a8c370aef24b
---
## Repo
`khursheedkhanaiforgood-ai/5320-onboarding`

## Pages source branch
**`main`** — NOT `gh-pages` (despite the branch existing).

The `gh-pages` branch exists in the repo but is NOT what serves the live URLs. It functions as a backup / staging area only. **Pushes to `gh-pages` alone do NOT publish to the live site.**

## To publish a new EOD HTML
1. Add the file to `docs/` on `feature/auto-deploy-agent` (development branch)
2. Cherry-pick or merge to `main` — Pages auto-builds from main
3. Update all three landing pages (parity convention) to add a card linking to the new EOD:
   - `index.html`
   - `index-nyt.html`
   - `index-harpers.html`
4. Push main → live URL becomes 200 within 1-3 minutes

## Live URL pattern
- Root: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/
- EOD HTML: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/session_summary_YYYYMMDD.html
- Resource files: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/data/<topic>/<file>

## What I got wrong May 7 2026
Spent ~30 minutes pushing to `gh-pages` and triggering empty commits / `.nojekyll` thinking the Pages branch was `gh-pages`. URLs were 404 the entire time. The other parallel session diagnosed it and fixed via commit `dcccb27` on `main` (cherry-picked the May 7 EOD + linked-resource folder + updated 3 landing pages).

**Lesson:** Always check `gh repo view --json defaultBranch,homepageUrl` or the Pages settings BEFORE assuming which branch publishes. The branch named `gh-pages` is not always the publishing branch.

## Index page parity convention
All 3 landing pages (`index.html`, `index-nyt.html`, `index-harpers.html`) must show the same set of EOD cards. When adding a new EOD, update all three. Per user feedback: "no material should repeat — just one entry point per resource" but cards across the 3 themes are not duplication — they're theme variants of the same entry.

## Cross-references
- Memory: `feedback_eod_html_format.md` — every session ends with EOD HTML pushed to GitHub Pages + linked from landing
- Memory: `project_index_dedup_pending.md` — known duplication of session_summary_20260422 link (deferred fix)