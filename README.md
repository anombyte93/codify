# codify

Refactor any Claude Code skill by extracting deterministic operations into a Python script. Reduces SKILL.md size by 60-80% while preserving all AI judgment.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/anombyte93/codify/main/install.sh | bash
```

This installs the skill to `~/.claude/skills/codify/`. Run the same command to upgrade.

**Check for updates:**
```bash
bash ~/.claude/skills/codify/install.sh --check-update
```

## What It Does

When you run `/codify` on a skill, it:

1. **Analyzes** the SKILL.md to identify deterministic vs judgment operations
2. **Extracts** deterministic logic (file checks, regex validation, math, file I/O) into a `script.py`
3. **Rewrites** SKILL.md to keep only AI judgment (questions, decisions, content generation)
4. **Tests** every subcommand produces valid JSON
5. **Verifies** the rewritten SKILL.md still contains all judgment content

## Architecture Principle

```
script.py = AI's calculator and file manager (deterministic tools)
SKILL.md  = AI's playbook for talking to user and making decisions (judgment)
```

## Example

Before codifying `prd-taskmaster`:
- SKILL.md: 1,342 lines (AI re-derives validation logic every run)

After:
- SKILL.md: 302 lines (judgment only)
- script.py: 1,079 lines (11 JSON-emitting subcommands)
- **77.5% reduction** in what the AI needs to read per invocation

## Requirements

- Claude Code CLI
- Python 3.8+

## License

MIT
