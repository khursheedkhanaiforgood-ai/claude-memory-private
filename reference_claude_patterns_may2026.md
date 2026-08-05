---
name: Claude Code Advanced Patterns — May 2026 Synthesis
description: Actionable patterns from official Claude Code best-practices docs, everything-claude-code framework, Taggart critique, Ordax LinkedIn series, Karpathy KB pattern, and Boris Cherny features. Synthesised May 15 2026.
type: reference
originSessionId: 6edeb609-0aad-4457-8d76-2accad2dd249
---
## Sources
- Claude Code official docs: code.claude.com/docs/en/best-practices
- Everything Claude Code: github.com/affaan-m/everything-claude-code (229 skills, hooks, cross-platform)
- Taggart: taggart-tech.com/reckoning/ (critical perspective, real usage patterns)
- Ordax LinkedIn series (eordax): .claude/ structure, 15 tips, Karpathy KB pattern
- Boris Cherny (creator) via Ordax: hidden features, voice coding, teleport

## Patterns Not Yet in Global CLAUDE.md

### Hooks vs CLAUDE.md — Critical Distinction
- CLAUDE.md = advisory. Claude can ignore it. Good for architecture and conventions.
- Hooks = guaranteed execution on every run. Use for: linting, test runs, security scans, anything that MUST always happen.
- Never rely on CLAUDE.md alone for safety-critical steps.

### `/compact "Focus on X"` — Directed Compaction
- Generic `/compact` preserves random context. Directed form preserves the specific thread.
- Use when approaching context limit mid-sprint: `/compact "Focus on XIQ→EP1 DB schema and agent routing logic"`

### Writer/Reviewer Subagent Pattern
- Two agents in sequence: Writer generates output (code, mapping, analysis); Reviewer gets fresh context with no knowledge of how it was generated.
- Fresh context = no anchoring bias. Writer can't critique its own work.
- Use for: code review, mapping validation, security auditing, any output where generator bias is a risk.
- One-liner: spawn a code-reviewer agent after every significant implementation.

### Fan-Out Loop Pattern (non-interactive)
```bash
# Template for batch processing
files=$(ls corpus/xiq/*.pdf)
for file in $files; do
  claude -p "extract nav paths from $file" --output-format stream-json >> results.jsonl
done
```
- `claude -p "prompt"` = non-interactive, scriptable, CI-compatible
- `--output-format stream-json` = parseable output for downstream processing
- Test on 2-3 items first, then scale to hundreds
- Perfect for: batch browser validation (31 bins × N features), doc ingestion pipeline

### File-Based State — FINDINGS.md Pattern
- For long/multi-session investigations: have Claude self-update a FINDINGS.md file
- Model is accountable across context windows — findings persist even after compaction
- Structure: date | finding | source | confidence | action-needed
- Direct application: XIQ→EP1 gap register, browser agent run logs

### ECC Framework Key Config
- `ECC_SESSION_START_CONTEXT=off` — disables auto-injection (saves tokens on focused sprints)
- `ECC_SESSION_START_MAX_CHARS=4000` — cap context injection
- Keep MCPs < 10 enabled at any time
- 229 skills available via `/ecc:plan`, `tdd-workflow`, `/code-review`, `/security-scan`
- Hook auto-loads from `hooks/hooks.json` (v2.1+) — don't add "hooks" field to plugin.json

## Behavioral Discipline — From Taggart + LinkedIn

### TDD as Hallucination Correction
- Write tests BEFORE asking Claude to implement. Once success criteria exist, Claude self-corrects on failures without human review.
- Without tests: you are the only feedback loop → fatigue → skipped scrutiny → silent errors in production.
- Applies even to non-code tasks: define what correct looks like FIRST.

### Role-Switching for Security
- Don't ask the implementing Claude to also security-audit.
- Separate pass: spawn security-reviewer agent with clean context. Different frame catches different vulnerabilities.

### Language Risk Profile
- Rust compile-time checks catch AI errors automatically — lower human policing cost.
- Python/JS/dynamic: AI errors pass compilation, require more review cycles. Different risk-benefit calculation.
- For production services with high stakes: strongly typed languages reduce AI hallucination blast radius.

### Human Review Fatigue is a Bug
- Repetitive approve-approve-approve → disengagement → missed critical changes.
- Mitigation: structure workflows so shortcuts are HARDER (require explicit justification), not easier.
- Counterintuitive: more automation → less approval fatigue → higher quality attention when it's needed.

### Expertise Decay Through Delegation
- Continuous babysitting of AI output prevents deep knowledge formation.
- The skill being preserved is "review AI output" not "build the thing".
- Counter: use Claude as sparring partner (co-thinking), not ghostwriter (full delegation).
- Specific practice: always understand the output before approving it.

### Mark AI-Generated Code in VCS
- Add AI attribution to commits (already done with Co-Authored-By line).
- In high-stakes code: comment blocks indicating AI-generated sections for traceability.

## Karpathy KB Pattern — Production Concerns

### For XIQ→EP1 Corpus Specifically:
1. **"Truth rot"** — XIQ docs move (v25.2.0 ≠ v25.9.0). Every doc in bibliography must carry `version` + `ingested_at`. Already in schema — enforce strictly. Never ingest without version tag.
2. **Immutable raw sources** — `corpus/xiq/` and `corpus/ep1/` raw files NEVER edited in-place. Corrections go to `corpus/corrections/` with source pointer.
3. **Audit trail** — RAG ingestion pipeline must log which doc version each chunk came from. `verified_by` in mappings table handles post-ingestion; pre-ingestion pipeline needs the same.
4. **Access control** — personal KB pattern ignores permission-based retrieval. For enterprise deployment, add org-level access control before sharing engine with other teams.

### General KB Workflow (Karpathy):
1. Ingest raw articles/papers into `raw/` — immutable
2. LLM builds `.md` wiki (summaries, backlinks, categories)
3. Query via LLM directly (~100 articles / ~400K words — no RAG needed at this scale)
4. Outputs (slides, charts) filed back into wiki
5. LLM linting: health-checks for inconsistencies, new connections

## .claude/ Structure — Ordax Insight
- Path globs on rule files = Claude loads only relevant rules for active file type
- This is why teams jump from ~60% to 95%+ instruction adherence
- Mechanism: not reading the whole ruleset every context load — surgical loading
- `.claude/rules/` files with YAML frontmatter + `globs: ["**/*.py"]` — load only for Python files

## Quick Reference — Boris Cherny Hidden Features
```
/teleport     Continue cloud session on local machine (or vice versa)
/loop         Run prompt on recurring schedule up to 1 week
/schedule     Schedule Claude at set interval up to 1 week
/batch        Fan out changesets to parallel worktree agents
/branch       Fork current session into new branch
--fork-session Fork from CLI: claude --resume <id> --fork-session
--bare        Speed up SDK startup 10x (skip heavy init)
--add-dir     Give Claude access to additional folder
--agent       Custom system prompt + tools for that session
-p "query"    Headless/CI mode — pipe Claude into scripts/pipelines
--output-format stream-json  Machine-readable streaming output
/voice        Enable voice input (Cherny's primary coding method)
Ctrl+B        Send current task to background, keep coding
Tab           Cycle: Normal → Auto-Accept → Plan modes
Shift+Tab     Alternate cycle
Double-Esc    Rewind menu — restore to any prior checkpoint
"ultrathink"  Force maximum reasoning depth
```
