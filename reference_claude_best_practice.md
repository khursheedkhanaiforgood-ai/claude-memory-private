---
name: Claude Code Best Practice Reference Repo
description: Reference repo for Claude Code configuration patterns — agents, skills, commands, hooks, settings
type: reference
---

GitHub repo: https://github.com/shanraisshan/claude-code-best-practice

A reference implementation (not an app) demonstrating best practices for Claude Code configuration. Covers:
- CLAUDE.md authoring (keep under 200 lines)
- Subagent definitions in `.claude/agents/*.md` with YAML frontmatter (name, description, model, tools, skills, hooks, permissionMode, maxTurns, memory, color)
- Skill definitions in `.claude/skills/<name>/SKILL.md` with YAML frontmatter
- Commands in `.claude/commands/*.md`
- `.claude/settings.json` for permissions, hooks, spinnerVerbs, plansDirectory
- `.mcp.json` for project-scoped MCP servers
- Hooks system for lifecycle events (PreToolUse, PostToolUse, Stop, etc.)
- Command → Agent → Skill orchestration pattern
- Two skill patterns: agent skills (preloaded via `skills:` field) vs invoked via `Skill` tool

Key advice from CLAUDE.md:
- Use Agent tool (not bash) to invoke subagents
- Keep CLAUDE.md under 200 lines per file
- Use commands for workflows, not standalone agents
- Compact at ~50% context usage
- Start with plan mode for complex tasks
