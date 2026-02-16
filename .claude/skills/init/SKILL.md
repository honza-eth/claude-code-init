---
name: init
description: Initialize any project with CLAUDE.md, STATE.md, JOURNAL.md, and verification hooks
user-invocable: true
argument-hint: "[project name]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# Project Initialization

Initialize this project with a structured scaffolding for effective AI-assisted development.

## STEP 1: Discovery

Ask these questions using AskUserQuestion:

**Free text (captures nuances):**
1. **Describe your project** — Goals, constraints, context, what makes it tricky. Allow free text via "Other" option.

**Multiple choice (ensures nothing missed):**
2. **Type?** (code / research / automation / docs)
3. **Duration?** (one-shot / multi-session / ongoing)
4. **Language/framework?** (if code)

Use `$ARGUMENTS` as the project name if provided, otherwise ask.

---

## STEP 2: Create Files

### 2a. CLAUDE.md — Copy EXACTLY (replace only `[Project Name]` and `[build/test commands]`):

```markdown
# [Project Name]

@STATE.md
@JOURNAL.md

---

## Principles

IMPORTANT: Always verify against actual sources. Never rely on training memory.
Read files before modifying. Check docs before using APIs. Explore before assuming.

### 1. Think First
State assumptions before acting. Say "I believe X because Y."
Present tradeoffs. Push back if a simpler approach exists. Useful > polite.

### 2. Minimal Changes
Only what was requested. No refactoring, no "improvements."

### 3. Surgical Edits
Change only necessary lines. Match existing style exactly.

### 4. Verify Everything
Convert tasks to testable goals. Step → verify → step → verify.

---

## Workflow

### Before ANY Change
1. Read relevant files
2. Check JOURNAL.md for prior decisions
3. State plan with verification steps

### After ANY Change
1. Verify it works
2. Update STATE.md (progress, open items)
3. Append to JOURNAL.md (findings, decisions)

### Context Management
- /clear between unrelated tasks
- After two failed corrections: /clear, write a better prompt

### Compaction
When compacting, always preserve: modified files list, verification commands, current phase from STATE.md.

---

## Session Start

1. Read STATE.md (where you left off)
2. Read JOURNAL.md (what's been learned)
3. Continue from last checkpoint
```

### 2b. STATE.md — Generate based on discovery:

```markdown
# Project State

## Current Phase
Just initialized.

## Done
- Project scaffolding created

## In Progress
- [First task based on discovery answers]

## Open Items
- [Any ideas or questions from discovery to revisit later]

## Blockers
None
```

### 2c. JOURNAL.md — Initialize with setup entry:

```markdown
# Journal

Append-only. Never edit previous entries.

---

## [Today's Date] - Project Setup

**Context**: Initialized project scaffolding.

**Findings**:
- [Initial observations about the codebase if existing code]
- [Key constraints or decisions from discovery]

**Decision**: [Project approach decided during discovery]
```

### 2d. .claude/rules/core.md — Permanent reinforcement (team-shared, checked into git):

```markdown
# Core Rules (Reinforced)

These rules are critical. Follow them without exception.

1. IMPORTANT: Always verify against actual sources. Read files, check docs, explore the codebase. Never rely on training memory. The codebase is the source of truth.
2. ALWAYS read files before modifying them.
3. ONLY make changes that were requested. No extras, no refactoring, no "improvements."
4. ALWAYS update STATE.md and append to JOURNAL.md after completing work.
```

### 2e. MEMORY.md — Seed auto-memory with critical rules:

Write to the project's auto-memory file at `~/.claude/projects/<project-path>/memory/MEMORY.md`.
Derive `<project-path>` from the current working directory by replacing `/` with `-` and stripping the leading `-`.

```markdown
# Project Memory

## Critical Rules (do not remove)
- ALWAYS verify against actual sources. Never rely on training memory.
- ALWAYS read files before modifying them.
- ONLY make changes that were requested.
- ALWAYS update STATE.md and append to JOURNAL.md after completing work.

## Project Notes
- Initialized with /init on [Today's Date]
```

Keep this under 30 lines so Claude has room to add its own notes (200-line limit).

---

## STEP 3: Verification Hook (Required for Code Projects)

Auto-detect the verification command by checking project files:

| If you find... | Use |
|----------------|-----|
| `package.json` with test script | `npm test \|\| true` |
| `pytest.ini` or `tests/` folder (Python) | `pytest \|\| true` |
| `go.mod` | `go build ./... \|\| true` |
| `Cargo.toml` | `cargo check \|\| true` |
| `Makefile` with test target | `make test \|\| true` |
| Formatter config (prettier, black, etc.) | Run formatter |
| Nothing found | Ask user or skip with note |

Create `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "[auto-detected verification command] || true"
          }
        ]
      }
    ],
    "PreCompact": [
      {
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/archive-context.sh"
          }
        ]
      }
    ]
  }
}
```

The `|| true` prevents hook failure from blocking work while still showing output.

---

## STEP 4: Create Archive Hook

Create `.claude/hooks/archive-context.sh` and make it executable:

```bash
#!/bin/bash
# Archives full conversation transcript before compaction

INPUT=$(cat)
TRANSCRIPT_PATH=$(echo "$INPUT" | jq -r '.transcript_path')
SESSION_ID=$(echo "$INPUT" | jq -r '.session_id')

ARCHIVE_DIR=".claude/archives"
mkdir -p "$ARCHIVE_DIR"

if [ -n "$TRANSCRIPT_PATH" ] && [ -f "$TRANSCRIPT_PATH" ]; then
  cp "$TRANSCRIPT_PATH" "$ARCHIVE_DIR/$(date +%Y%m%d-%H%M%S)-${SESSION_ID:0:8}.jsonl"
fi

# Guide compactor on what to preserve
echo '{"hookSpecificOutput":{"hookEventName":"PreCompact","customInstructions":"Preserve: modified files list, current STATE.md status, verification commands, and all open items."}}'

exit 0
```

Run `chmod +x .claude/hooks/archive-context.sh` after creating it.

---

## Rules for This Bootstrap

- Copy CLAUDE.md template exactly (only replace `[Project Name]` and fill STATE/JOURNAL from discovery)
- Keep CLAUDE.md under 50 lines
- Use "always" / "only" — never "when appropriate" or "consider"
- Every rule traces to a source: Karpathy (principles), Vercel (passive context), Cherny (verification), Anthropic (emphasis markers)
