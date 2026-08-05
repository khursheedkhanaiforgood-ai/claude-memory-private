---
name: 5320-onboarding GitHub Pages Config
description: GitHub Pages for khursheedkhanaiforgood-ai/5320-onboarding serves from `main` branch root. The `gh-pages` branch exists in parallel but is NOT deployed. EODs must land on main + update 3 index pages.
type: reference
originSessionId: 52c71221-9cc9-44db-9c54-689b1e8f03fe
---
## Repo
`khursheedkhanaiforgood-ai/5320-onboarding`
Live: https://khursheedkhanaiforgood-ai.github.io/5320-onboarding/

## Pages source
**`main` branch root** — confirmed May 7 2026 by curl 200 vs 404 testing. `gh-pages` branch exists with parallel content but is NOT served by Pages. Pushing EOD HTML only to `gh-pages` will result in a 404 on the live URL.

## Confirmed by elimination (May 7 2026)
- `session_summary_20260422.html` exists only on `main` → returns 200 ✓
- `session_summary_20260507.html` existed only on `gh-pages` → returned 404 ✗
- After cherry-pick to `main` (commit `dcccb27`) + Pages rebuild → returns 200 ✓

## Convention for new EODs (per existing pattern)
1. Push `session_summary_YYYYMMDD.html` and any `data/*` artifacts to **`main` branch root** (not docs/, not gh-pages)
2. Update **all three** landing pages for parity: `index.html`, `index-nyt.html`, `index-harpers.html`
3. Add an EOD card to the "EOD Blueprints" grid in each
4. If there's a session log, add a card to the "Full Session Logs" grid in each
5. Verify URL returns 200 after Pages rebuild (~1-2 min)

## Index card style ledger (so future EODs match)
| Date | index.html border | index-nyt border | index-harpers border-left |
|------|-------------------|-------------------|----------------------------|
| Apr 20 | #7c3aed (purple) | default | default |
| Apr 21 | #0891b2 (cyan) | default | default |
| Apr 22 ★ Blueprint | #16a34a (green) | #326891 | default |
| Apr 23 ★ Review | #0891b2 | #326891 | default |
| Apr 28 ★ Debug | #4ade80 | #166534 | #C41A0F |
| Apr 30 ★ VOSS | #ea580c (orange) | #ea580c | #ea580c |
| May 1 ★ IPE | #0891b2 (cyan) | #326891 | #326891 |
| May 4 ★ EAPOL | #db2777 (pink) | #9d174d | #9d174d |
| May 7 ★ DHCP | #dc2626 (red) | #991b1b | #991b1b |

## Why gh-pages exists
Earlier sessions pushed EODs to `gh-pages` thinking it was the deployment branch (standard GitHub convention). It became a parallel/backup. Don't delete — it has commit history. But for new work, push to `main`.

## Cross-refs
- `feedback_eod_html_format.md` — EOD HTML format requirements (already says "main branch")
- `project_session_20260507.md` Workstream 1c — full record of the May 7 fix
