# /init — Claude Code Project Bootstrap Skill

A skill that scaffolds any project for effective AI-assisted development.

## Install

```bash
cp -r .claude/skills/init ~/.claude/skills/init
```

## Use

In any project:

```
/init my-project-name
```

## What It Creates (6 files + git/GitHub)

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Principles + workflow (~50 lines). Auto-loaded every session. |
| `STATE.md` | Where we are: current phase, done, in progress, open items, blockers. Auto-loaded via `@import`. |
| `JOURNAL.md` | What we've learned: research, decisions, findings. Append-only. Read on-demand. |
| `.claude/rules/core.md` | Critical rules doubled — in CLAUDE.md and here — harder to ignore. Auto-loaded. |
| `.claude/settings.json` | PostToolUse hook (auto-runs tests after edits) + PreCompact hook (archives context). |
| `.claude/hooks/archive-context.sh` | Saves full conversation transcript before context compaction. |

## The 3 Principles

1. **Think First** — State assumptions, present tradeoffs, push back if simpler exists
2. **Minimal, Surgical Changes** — Only what was requested, change only necessary lines
3. **Verify Everything** — Step → verify → step → verify

## Anti-Lazy-Agent Rule

> IMPORTANT: Always verify against actual sources. Never rely on training memory.

Double-injected: appears in CLAUDE.md and `.claude/rules/core.md`. Both auto-loaded, so the rule is in context twice every session.

## Sources

- [Karpathy Agent Critique](https://x.com/karpathy/status/2015883857489522876)
- [Boris Cherny — How Boris Uses Claude Code](https://howborisusesclaudecode.com/)
- [Vercel Research](https://vercel.com/blog/agents-md-outperforms-skills-in-our-agent-evals)
- [Anthropic Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [HumanLayer — Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
