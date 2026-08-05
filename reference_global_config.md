---
name: Global Claude Config Repo
description: Synthesized global Claude Code standards from 16 sources — pushed to GitHub 2026-04-06. Applies to ALL projects.
type: reference
---

# Global Claude Config Repo

**GitHub:** https://github.com/khursheedkhanaiforgood-ai/claude-global-config  
**Local mirror:** `~/.claude/CLAUDE.md` (updated 2026-04-06)  
**Synthesized from:** 16 sources on 2026-04-06

## Key New Patterns Added (beyond previous config)

### Token Optimization (Eduardo)
- Edit not resend → 80-90% token savings
- Fresh context every 15-20 messages
- Model routing: Haiku→quick, Sonnet→real work, Opus→hard problems
- /context + /compact as primary token management tools

### Power Commands (Boris Cherny, creator)
- /teleport, /loop, /schedule, /btw, /batch, /rewind, /chrome
- --bare (10× SDK startup), --add-dir, --agent
- Shift+Tab (mode cycling), "Ultra think" (max reasoning)

### Architecture Patterns (Eduardo anatomy)
- rules/ folder with YAML frontmatter scoping (new — not in old config)
- skills/ as auto-triggered packages (new)
- CLAUDE.md ≤ 200 lines constraint (now enforced)

### everything-claude-code (affaan-m)
- Install: `./install.sh --profile full` → 38 agents, 156 skills
- Continuous learning: /learn → /evolve → /prune
- Hook profiles: ECC_HOOK_PROFILE=minimal|standard|strict
- Verification loops: /verify, /checkpoint, /quality-gate

### Karpathy Knowledge Base Pattern
- raw/ → wiki/ → Q&A → outputs → linting
- Maps directly to cisco-en-cli-agent RAG pipeline

### Quality Gates (Taggart)
- TDD mandatory, security audit before endpoints, scope narrowly

## Repo Structure
```
claude-global-config/
├── CLAUDE.md
├── synthesis/
│   ├── 01-token-optimization.md
│   ├── 02-claude-code-power-commands.md
│   ├── 03-architecture-patterns.md
│   ├── 04-knowledge-base-pattern.md
│   └── 05-quality-gates.md
└── references/
    ├── cisco-ios-xe.md
    └── stanford-cs221.md
```
