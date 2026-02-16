---
name: init
description: Initialize any project with CLAUDE.md, STATE.md, JOURNAL.md, and verification hooks
user-invocable: true
argument-hint: "[project name]"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# Project Initialization

Initialize this project with structured scaffolding for effective AI-assisted development.

## STEP 0: Check for Existing Files

Before creating anything, check if CLAUDE.md, STATE.md, JOURNAL.md, or .claude/settings.json already exist.
If any exist, ask the user: **Overwrite, merge, or abort?**
Never silently overwrite existing files.

---

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

### 2a. CLAUDE.md — Copy EXACTLY (replace only `[Project Name]`):

STATE.md is `@`-imported (auto-loaded, small and mutable). JOURNAL.md is read on-demand via Session Start (grows over time, would bloat context if auto-loaded).

```markdown
# [Project Name]

@STATE.md

---

## Principles

IMPORTANT: Always verify against actual sources. Never rely on training memory.
Read files before modifying. Check docs before using APIs. Explore before assuming.

### 1. Think First
State assumptions before acting. Say "I believe X because Y."
Present tradeoffs. Push back if a simpler approach exists. Useful > polite.

### 2. Minimal, Surgical Changes
Only what was requested. Change only necessary lines. Match existing style.

### 3. Verify Everything
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

### 2d. .claude/rules/core.md — Double-injection of critical rules:

These must include all Principles from CLAUDE.md plus the STATE.md/JOURNAL.md workflow rule.

```markdown
# Core Rules (Reinforced)

These rules are critical. Follow them without exception.

1. IMPORTANT: Always verify against actual sources. Never rely on training memory. Read files before modifying. Check docs before using APIs. Explore before assuming.
2. State assumptions before acting. Present tradeoffs. Push back if a simpler approach exists.
3. Only what was requested. Change only necessary lines. Match existing style.
4. Convert tasks to testable goals. Step → verify → step → verify.
5. ALWAYS update STATE.md and append to JOURNAL.md after completing work.
```

---

## STEP 3: Verification Hook (Required for Code Projects)

Auto-detect the verification command by checking project files:

| If you find... | Use |
|----------------|-----|
| `package.json` with test script | `npm test` |
| `pytest.ini` or `tests/` folder (Python) | `pytest` |
| `go.mod` | `go build ./...` |
| `Cargo.toml` | `cargo check` |
| `Makefile` with test target | `make test` |
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
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/archive-context.sh"
          }
        ]
      }
    ]
  }
}
```

The `|| true` prevents hook failure from blocking work while still showing output.

Prefer fast checks (linter, type-checker) over slow test suites — this hook runs on every file edit.

---

## STEP 4: Create Archive Hook

Create `.claude/hooks/archive-context.sh` and make it executable:

```bash
#!/bin/bash
# Archives full conversation transcript before compaction

INPUT=$(cat)

# Parse transcript path — try jq first, fall back to grep
if command -v jq &> /dev/null; then
  TRANSCRIPT_PATH=$(echo "$INPUT" | jq -r '.transcript_path')
  SESSION_ID=$(echo "$INPUT" | jq -r '.session_id')
else
  TRANSCRIPT_PATH=$(echo "$INPUT" | grep -o '"transcript_path":"[^"]*"' | cut -d'"' -f4)
  SESSION_ID=$(echo "$INPUT" | grep -o '"session_id":"[^"]*"' | cut -d'"' -f4)
fi

ARCHIVE_DIR="$CLAUDE_PROJECT_DIR/.claude/archives"
mkdir -p "$ARCHIVE_DIR"

if [ -n "$TRANSCRIPT_PATH" ] && [ -f "$TRANSCRIPT_PATH" ]; then
  cp "$TRANSCRIPT_PATH" "$ARCHIVE_DIR/$(date +%Y%m%d-%H%M%S)-${SESSION_ID:0:8}.jsonl"
fi

exit 0
```

Run `chmod +x .claude/hooks/archive-context.sh` after creating it.

Note: `transcript_path` may be empty on some systems (known bug). The script handles this gracefully — if empty, no archive is created.

---

## STEP 5: Git + GitHub

1. If not already a git repo, run `git init` and rename branch to `main`.
   If already a git repo, skip to step 2.
2. Create `.gitignore` with at minimum:
   ```
   .DS_Store
   .claude/settings.local.json
   CLAUDE.local.md
   .claude/archives/
   ```
   If `.gitignore` already exists, append any missing entries.
3. Stage all created files and commit:
   ```
   git add .gitignore CLAUDE.md STATE.md JOURNAL.md .claude/
   git commit -m "Initial project scaffolding via /init"
   ```
4. Create a GitHub repo using `gh`:
   - Use the project name from discovery as the repo name
   - Ask the user: **Public or private?**
   - `gh repo create [project-name] --[public|private] --source=. --push`
5. Confirm the repo URL to the user.

If `gh` is not installed or not authenticated, warn the user and skip the GitHub step.
