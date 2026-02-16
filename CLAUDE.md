# Claude Code Meta-Framework

The `/init` skill bootstraps any project with structured scaffolding for AI-assisted development.

## Project Structure

- `.claude/skills/init/SKILL.md` — The skill. This is the only file that matters.
- `README.md` — Public documentation.
- `CLAUDE.md` — This file.

## What /init Creates (6 files + git/GitHub)

| File | Purpose | Nature |
|------|---------|--------|
| CLAUDE.md | Principles + workflow (~50 lines) | Fixed template, auto-loaded |
| STATE.md | Phase, done, in progress, open items, blockers | Mutable snapshot, auto-loaded via @import |
| JOURNAL.md | Research, decisions, findings | Append-only, read on-demand |
| .claude/rules/core.md | Critical rules (double-injection) | Fixed, auto-loaded |
| .claude/settings.json | PostToolUse verification + PreCompact archive hooks | Auto-detected |
| .claude/hooks/archive-context.sh | Saves full transcript before compaction | Fixed |

## Working on SKILL.md

All changes should:

1. Keep the CLAUDE.md template inside it under 50 lines
2. Follow the principles it teaches (simplicity, surgical changes)
3. Use directive language ("always", "must") not suggestive ("if needed", "consider")
4. Prefer retrieval-led reasoning — read files before modifying, never assume from training data

## Design Principles

| Principle | Source |
|-----------|--------|
| Dumber > Smarter | Vercel research |
| Less is More | Boris Cherny |
| Passive > Active | Vercel research |
| Fixed > Generated | Project design choice |
| Double-injection | Reinforces critical rules via CLAUDE.md + rules/core.md |
| Verify everything | Boris Cherny, Anthropic |
