# Claude Code Memory — Private

Persistent memory system for Claude Code sessions. Contains session context, project state, feedback rules, and references accumulated across all work sessions.

## Structure

| File | Purpose |
|------|---------|
| `MEMORY.md` | Session-load index — loaded on every Claude Code startup |
| `memory_archive_dormant.md` | Archive of stale/completed entries removed from MEMORY.md to stay under size limit |
| `feedback_*.md` | Behavioral rules and corrections from past sessions |
| `project_*.md` | Active and historical project state files |
| `reference_*.md` | Pointers to external resources, CLI references, runbooks |

## Excluded Files (not in this repo)

The following two files were excluded from version control because they contain live credentials (plaintext app passwords and a GitHub PAT fragment). They remain on disk at the local memory path only.

| File | Reason excluded |
|------|----------------|
| `project_cisco_en_cli_agent.md` | Contains plaintext APP_USERS passwords (`Extreme01!!`, `dhaka1234`) and a partial GitHub PAT (`ghp_gLforg5V...`) |
| `security_assessment_cisco_en_cli_agent.md` | Contains the same partial GitHub PAT in the security findings log |

These are enforced by `.gitignore` in this repo.

## Usage

Memory is read automatically at every Claude Code session start. To push updates:

```bash
cd ~/.claude/projects/-Users-khukhan/memory
git add -A && git commit -m "memory update $(date +%Y-%m-%d)" && git push
```
