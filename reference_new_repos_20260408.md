---
name: New Claude Code Reference Repos — April 8 2026
description: Additional repos and links added from user's docx: claude-mem, career-ops, Boris Cherny X post, Taggart critical perspective
type: reference
---

## claude-mem (thedotmack)
https://github.com/thedotmack/claude-mem
Memory persistence system for Claude Code — custom hooks that automatically save and load context across sessions. Use for long-running projects where context continuity is critical.

## Boris Cherny Hidden Features (X post)
https://x.com/bcherny/status/2038454336355999749
Creator's own list of hidden/underused features. Key additions:
- `/voice` — voice input (Boris does most coding by speaking)
- `/branch` and `claude --resume <id> --fork-session` — fork sessions
- `/schedule` — schedule Claude to run at intervals (up to 1 week)
- `Ctrl+B` — background tasks
- `--add-dir` — give Claude access to more folders
- `--agent` — custom system prompt + tools per session
- `/effort low|high` — match reasoning depth to task

## Taggart Critical Perspective
https://taggart-tech.com/reckoning/
Michael Taggart's experienced software engineer critique of AI coding tools. Key concern: volume/velocity pressure leads to unmaintainable code ("most brute force, messiest way to do X"). Important counterbalance — use AI tools for leverage, not as a replacement for architecture thinking.

## ninadurann Agentic Workflows Repo
https://lnkd.in/eDXXN6wt
Command → Agent → Skill orchestration pattern. Parallel dev with Agent Teams + Git Worktrees.
Debugging with /doctor and MCP (Playwright, Chrome DevTools). Sub-agents with persistent identity.
Custom status lines to monitor cost + session info.

## GeekWire — AI Coach vs Ghostwriter (2026)
https://www.geekwire.com/2026/opinion-ai-coach-or-ai-ghostwriter-the-choice-is-yours/
Counterbalance to Taggart. Frames the choice: use AI to learn and grow (coach) vs replace thinking (ghostwriter). Relevant for how Khursheed uses Claude Code for lab learning.

## LinkedIn Posts (aggregated April 2026)
Synthesized into CLAUDE.md updates:
- Token pacing: 5-hour rolling window, 2-3 sessions → 150-200+ messages/day
- Skills as folders (`.claude/skills/<name>/`) with SKILL.md frontmatter
- `/doctor` for debugging Claude Code setup
- Status lines to monitor cost + session info
- Headless/CI mode: `claude -p "query" --output-format json`
- Claude mobile app — code by voice from phone
- Cowork Dispatch — manage Slack/email/files when away from computer
- Chrome extension — frontend verification loop (Claude iterates until UI is correct)
- Claude Desktop — auto-starts + tests local web servers
- Projects cache recurring files (PDFs, docs) — no re-upload each session
- Tab cycles: Normal → Auto-Accept → Plan
