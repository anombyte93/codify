---
name: codify
description: "Refactor any skill by extracting deterministic operations into a Python script and slimming the SKILL.md to only AI judgment parts. Use when user says /codify, codify a skill, or optimize a skill."
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - AskUserQuestion
---

# Codify Skill

> `/codify <skill-name>` — Extract deterministic logic from a skill's SKILL.md into a Python script.

## What This Does

Takes a skill's SKILL.md and splits it into:
- **Script** (`<skill-dir>/script.py`) — all deterministic operations (file ops, validation, parsing, template copying)
- **Slim SKILL.md** — only AI judgment parts (questions, assessment, continuation logic) + calls to the script

## Argument

`/codify <skill-name>` — the skill directory name under `~/.claude/skills/`. If omitted, list available skills and ask.

## Workflow

### Phase 1: Analyze

1. Read `~/.claude/skills/<skill-name>/SKILL.md`
2. Classify every block of instructions into one of two buckets:

**DETERMINISTIC** (goes into script.py):
- File/directory creation, copying, moving, deleting
- Template validation and repair
- Reading files and extracting sections/data
- Writing structured content to files
- Git operations (status, branch, mv)
- Environment detection (checking if files/dirs exist)
- Any operation that always produces the same output given the same input
- Pattern matching, categorization by filename/extension
- JSON/YAML/TOML parsing or generation
- Archiving, rotating, or resetting files

**JUDGMENT** (stays in SKILL.md):
- Asking the user questions (AskUserQuestion)
- Deciding what to do based on context (if/else logic about user intent)
- Assessing completion status
- Choosing what to work on next
- Dispatching agents for tasks requiring reasoning
- Evaluating quality (e.g., "is this content worth promoting?")
- Any operation where different AI instances might reasonably make different choices

3. Present the analysis to the user:

```
SKILL: <name> (<line-count> lines)

DETERMINISTIC operations found:
- <description of each operation that will become a script subcommand>

JUDGMENT operations (staying in SKILL.md):
- <description of each AI reasoning block>

Proposed script subcommands:
- <subcommand-name>: <what it does>
- <subcommand-name>: <what it does>
```

Ask user to approve or adjust before proceeding.

### Phase 2: Build Script

Create `~/.claude/skills/<skill-name>/script.py` following these rules:

**Script Architecture**:
- argparse with subcommands (one per deterministic operation)
- ALL output is JSON via a shared `_out(data)` helper
- Handle markdown code blocks correctly when parsing sections (track ``` state)
- Use pathlib for all file operations
- Include `--help` for every subcommand
- Make executable (`chmod +x`)
- Use `#!/usr/bin/env python3`
- No external dependencies — stdlib only

**Subcommand Design**:
- Each subcommand does ONE thing well
- Required args via `--flag` (not positional)
- Return `{"status": "ok", ...}` on success
- Return `{"status": "error", "message": "..."}` and exit(1) on failure
- Include enough detail in output for AI to make decisions without re-reading files

**Code Quality**:
- Proper error handling (file not found, permission denied)
- Idempotent where possible (safe to run twice)
- Comments only where logic is non-obvious

### Phase 3: Rewrite SKILL.md

Rewrite the SKILL.md keeping:
- Frontmatter (name, description, user-invocable) — unchanged
- UX contracts, invariants, rules — preserved but condensed
- User question definitions — preserved
- Continuation/decision logic — preserved
- Self-assessment criteria — preserved

Replace with script calls:
- Every deterministic operation becomes a code block showing the script call
- Agent Ref / Agent Reference sections → deleted entirely
- Inline bash snippets → replaced with script subcommand calls
- Multi-step file operations → single script subcommand

Preserve and update frontmatter:
- Keep `name`, `description`, `user-invocable` unchanged
- **Add `allowed-tools`** if missing — list tools the skill actually uses (Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, etc.)
- Update `description` if the workflow changed significantly

Add a Script Reference table at the bottom:

```markdown
## Script Reference

All commands output JSON. Run from skill directory.

| Command | Purpose |
|---------|---------|
| `subcommand-1 --flag` | What it does |
| `subcommand-2 --flag` | What it does |
```

**Target**: Reduce SKILL.md by 50-70% while preserving all behavior.

### Phase 4: Test

Run each script subcommand with `--help` to verify it parses.
Run `preflight` or equivalent read-only command against the current project to verify JSON output.
Report results.

### Phase 5: Backup

Before overwriting SKILL.md, save original as `SKILL.md.pre-codify`.

## Edge Cases

- **Skill has no deterministic operations** (pure judgment): Report "Nothing to extract" and skip.
- **Skill already has a script.py**: Ask whether to merge, replace, or abort.
- **Skill is very short (<50 lines)**: Report "Too small to benefit from codification" and skip.
- **SKILL.md references other files**: Read those too and include in analysis.

## Example

```
/codify start
```

Result: The transformation we just did — 637-line SKILL.md with 7 Agent Ref sections became 204-line SKILL.md + 320-line session-init.py script with 9 subcommands.
