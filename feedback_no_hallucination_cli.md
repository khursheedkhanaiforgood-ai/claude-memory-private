---
name: No CLI Hallucination — Never Guess Commands
description: Do not suggest CLI commands unless 100% certain they are correct. Never experiment with syntax on live devices without user permission.
type: feedback
originSessionId: 2685150a-5580-49b4-804b-fd57b6be1569
---
Do not suggest CLI commands unless 100% certain of the exact syntax.

**Why:** Guessing commands on live AP/switch CLI wastes time, erodes trust, and can cause unintended config changes. June 12 2026: wasted multiple rounds guessing ADSP disable syntax on IQ Engine — none worked.

**How to apply:**
- If unsure, stop immediately and say "I don't know."
- Do not suggest alternatives, do not chain guesses, do not offer workarounds.
- Just say "I don't know" and wait for the user to direct next steps.
- Only experiment if the user explicitly asks for it.
- This applies to ALL CLI environments and ALL technical domains.
