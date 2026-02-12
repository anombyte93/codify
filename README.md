# /codify

[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-8A2BE2)](https://github.com/anombyte93/codify)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/anombyte93/codify/blob/main/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/anombyte93/codify)](https://github.com/anombyte93/codify/stargazers)
[![GitHub last commit](https://img.shields.io/github/last-commit/anombyte93/codify)](https://github.com/anombyte93/codify/commits)

> Refactor any Claude Code skill by extracting deterministic operations into a Python script.

Every time a skill runs, the AI re-reads hundreds of lines of prose to re-derive mechanical operations: "does this file exist?", "run this regex", "copy with timestamp". This is wasteful. **codify** splits skills into two parts: code for mechanics, prompts for judgment.

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/anombyte93/codify/main/install.sh | bash
```

Installs to `~/.claude/skills/codify/`. Run the same command to upgrade.

```bash
# Check for updates
bash ~/.claude/skills/codify/install.sh --check-update
```

## Usage

```
/codify <skill-name>
```

Where `<skill-name>` is the directory name under `~/.claude/skills/`.

## How It Works

### Phase 1: Analyze

Reads the skill's SKILL.md and classifies every block:

| Category | Goes to | Examples |
|----------|---------|----------|
| **Deterministic** | `script.py` | File existence checks, regex validation, math, template loading, file copying, JSON parsing |
| **Judgment** | `SKILL.md` | User questions, decisions, content generation, quality assessment, continuation logic |

### Phase 2: Build script.py

Creates a Python script with argparse subcommands. Every subcommand:
- Does one thing well
- Outputs JSON to stdout
- Responds to `--help`
- Uses stdlib only (no dependencies)

### Phase 3: Rewrite SKILL.md

Replaces deterministic prose with script calls:

```markdown
# Before (in SKILL.md)
Check if .taskmaster/ exists. If it does, look for PRD files
in .taskmaster/docs/. Read tasks.json to count tasks...
(~40 lines of detection logic)

# After (in SKILL.md)
python3 script.py preflight
# Returns JSON: has_taskmaster, prd_path, task_count, ...
```

### Phase 4: Test & Backup

- Runs `--help` on every subcommand
- Runs read-only commands to verify JSON output
- Saves original as `SKILL.md.pre-codify`

## Real Results

| Skill | SKILL.md Before | SKILL.md After | Reduction | script.py |
|-------|----------------|----------------|-----------|-----------|
| prd-taskmaster | 1,342 lines | 302 lines | **77.5%** | 1,079 lines (11 subcommands) |
| start | 637 lines | 204 lines | **68.0%** | 320 lines (9 subcommands) |

## Architecture

```
script.py = AI's calculator and file manager (deterministic tools)
SKILL.md  = AI's playbook for user interaction and decisions (judgment)
```

The AI calls script.py like any other tool. Script returns structured JSON. AI interprets the results and makes decisions. Neither replaces the other.

## Requirements

- [Claude Code](https://claude.ai/code) CLI
- Python 3.8+

## License

MIT - see [LICENSE](LICENSE)

---

Built by [Atlas AI](https://atlas-ai.au)