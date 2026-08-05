---
name: Sprint C EP1 Traverser — session 2026-05-28
description: Sprint C Playwright scraper status, xcloud parcel blocker root cause, files changed, next-session investigation plan
type: project
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
Sprint C scraper scaffold is COMPLETE but xcloud bins are BLOCKED by `ws-routed-parcel` never mounting content after org selection.

**Files created / modified (2026-05-28):**
- `sprint_c/extractor.py` — field extractor: 14 en-* component types, shadow-DOM JS evaluate, label resolution (7-step), NOISE_PATTERNS for AG Grid column picker
- `sprint_c/traverser.py` — 34-bin traversal loop; `_get_parcel_state()` helper; rewritten `select_ep1_network()`; `navigate_to_bin()` with xcloud parcel check
- `sprint_c/bins.py` — all 34 bins status=CONFIRMED, all `/configuration/*` → `/xcloud/configuration/*`
- `sprint_c/config.py` — `ep1_network_name = "Extreme Networks"` (the `<a class="org-item">` text; "Home VIQ" was wrong — just a tooltip)

**The xcloud parcel blocker (still unresolved):**
- Clicking `a.org-item` ("Extreme Networks") → parcel drops to children=0
- Re-navigating to `/xcloud/...` → network selector appears AGAIN (org context not persisted)
- Loop repeats indefinitely; `RbacInfoService: roleMap is not initialized` ×18 warnings each time
- False positive bug (now fixed): empty parcel (children=0) was passing `"organization" not in text` check → returning True when actually broken
- Current `select_ep1_network` uses Playwright native `locator.click()` + re-navigation + 20s content wait — but selector still re-appears

**Hypotheses for next session (in priority order):**
1. Intercept the XHR/fetch request on `a.org-item` click via `page.on("request")` — confirm the org-selection API call actually fires and returns 200
2. Check `localStorage` / `sessionStorage` before and after click for org context keys (e.g. `orgId`, `networkId`, `selectedOrg`)
3. Try **"KB lab OH"** (visible under Other Organizations in apps panel) — "Extreme Networks" may not have xcloud licensed for this account
4. Try Angular router navigation (click sidebar "Configuration" link) instead of `page.goto()` — router guards may be required to initialize RBAC

**Monitoring bins (non-xcloud) not yet tested separately** — bins 1,3,5,21–25,32 use `/monitoring/*` routes which do NOT require org selection. These should work fine.

**Why:** The xcloud micro-frontend (single-spa SystemJS) bootstraps with `RbacInfoService`. If the org context cookie/token is not set properly after clicking org-item, every subsequent xcloud navigation re-triggers the network selector. This is likely an account-level config issue, not a code issue.

**How to apply:** Next session — run `--bins 21` (monitoring/dashboard, no xcloud) first to confirm monitoring bins work. Then investigate xcloud org persistence with the network-intercept approach above before spending more time on the selector loop.
