---
name: EOD HTML Format — NYT Style with Adaptive TOC
description: EOD HTML uses NYT journal format (white, serif, TOC-driven). Confirmed as preferred over fixed 8-slot dark nav. TOC adapts to what was done that day. Updated May 12 2026.
type: feedback
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Primary Format: NYT Journal Style
**Confirmed May 12 2026** — user prefers the May 11 EOD format over the May 12 dark dashboard format.
The May 11 format (`session_summary_20260511.html`) is the reference example.

## Why NYT over dark dashboard
- TOC is **content-first**: write the sections that fit the day, then link them. The fixed 8-slot nav forces shape before content.
- Editorial tone (italic subtitle, byline, h1 headline) makes entries feel like readable lab journal entries, not forms to fill.
- The `.toc` block at top of article IS the navigation — no sticky pinned nav required.
- Different sessions need different sections: a pure lab day ≠ a Socratic session ≠ a sprint planning day.

## Visual Standard (NYT Style)
```css
body        { background: #FFFFFF; color: #111111; font-family: 'Libre Baskerville', serif; }
.container  { max-width: 760px; margin: 0 auto; padding: 48px 24px; }
.nyt-logo   { font-family: 'Libre Franklin'; font-size: 0.62rem; font-weight: 700;
              text-transform: uppercase; letter-spacing: 0.22em; color: #666; text-align: center; }
.byline     { font-family: 'Libre Franklin'; font-size: 0.72rem; color: #666; }
h1          { font-size: 2.2rem; font-weight: 700; line-height: 1.15; }
.subtitle   { font-style: italic; color: #444; font-size: 1rem; line-height: 1.8; }
.section-label { font-family: 'Libre Franklin'; font-size: 0.62rem; font-weight: 700;
                 text-transform: uppercase; letter-spacing: 0.18em; color: #326891;
                 border-bottom: 2px solid #326891; }
.toc        { background: #F8F8F8; border: 1px solid #E2E2E2; padding: 18px 22px; }
.toc a      { font-family: 'Libre Franklin'; font-size: 0.85rem; color: #326891;
              display: block; border-bottom: 1px dotted #E2E2E2; padding: 4px 0; }
```

## Page Structure (fixed)
```
<hr class="nyt-rule-top">     ← thick 3px black line
<p class="nyt-logo">          ← "E2E Network Deployment — Lab Journal"
<hr class="nyt-rule-thin">    ← 1px line
<p class="byline">            ← date · time · lab context
<h1>                          ← editorial headline — what this session was about
<p class="subtitle">          ← italic one-paragraph summary of the arc
<div class="toc">             ← "In This Session" — clickable linked list
<div class="stats">           ← optional: 4-stat quick summary grid
[sections...]                 ← adaptive content sections
<p class="footer">            ← copyright
```

## TOC — Adaptive, Not Fixed
The TOC links adapt to whatever sections were covered. No two EODs have identical TOCs.

**Recurring soft defaults** (include when relevant, skip when not):
| Section | ID | When to include |
|---------|----|-----------------|
| Lab State / Assets | `#lab-state` | Any session with HW changes or new configs |
| Conceptual Corrections | `#corrections` | Any Socratic session with misconceptions resolved |
| Sprint Backlog | `#sprints` | When sprint decisions were made |
| Simulators | `#simulators` | When a simulator was built or updated |
| References | `#references` | When RFCs/standards were the focus |
| Pending | `#pending` | Always — what's left to do |

**Section labels** use `.section-label` in `#326891` blue with underline.
**Callout boxes**: `.box` (blue), `.box.warn` (amber), `.box.ok` (green), `.box.correct` (red — for misconception corrections).

## Stats Bar (optional)
Use when the session has quantifiable deliverables:
```html
<div class="stats">
  <div class="stat-card"><div class="stat-num">4</div><div class="stat-label">Principles</div></div>
  ...
</div>
```

## Overridden guidance
The earlier rule (dark-themed dashboard, 8-slot sticky nav: Sprint | Arch | Socratic | Simulator | Phase | Lab State | Pending | URLs) is **superseded** for narrative session EODs.

Dark theme is still appropriate for **pure reference pages** (credential sheets, command quick-reference) — not for session narrative EODs.

## Also applies to presentation decks and diagnostic write-ups
Confirmed 2026-08-10: any HTML deck or forensic write-up (e.g. `deck_8021x_vlan_diagnosis.html`) must also use the NYT journal template, NOT a dark slide-style theme. The sticky TOC nav acts as slide navigation. Sections replace slides.

**Why:** User corrected dark-themed deck on GitHub Pages — "This is NOT the NYT template." Apply NYT universally to all 5320-onboarding HTML outputs.

## Canonical reference
`/Users/khukhan/5320-onboarding-agent/docs/session_summary_20260807.html` — use this as the CSS/structure baseline for all new HTML files in this project.

## How to apply
1. Write the headline first — what was the defining arc of this session?
2. Write the italic subtitle — one paragraph summary of the narrative.
3. List the sections that were actually covered → build TOC from those.
4. Never add a section just because it was in a previous EOD's TOC.
5. For decks: sections = slides. Sticky TOC = slide nav. Same CSS, no exceptions.
