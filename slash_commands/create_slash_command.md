Slash command to generate properly structured slash command files for Claude Code with standardized structure

💡 Use Case: Use when creating custom slash commands for repetitive tasks or project-specific workflows

---

description: Create new slash commands with standardized structure
argument-hint: [name] [purpose]
allowed-tools: Bash(mkdir:_), Bash(tee:_), Bash(test:\*)

---

## /create-command

## Purpose

Generate properly structured slash command files for Claude Code.

## Contract

**Inputs:**

- `$1` — COMMAND_NAME (lowercase, kebab-case, verb-first)
- `$2` — PURPOSE (command description)

**Outputs:**

- `STATUS=<WROTE> PATH=<path>`

## Instructions

1. **Validate inputs:**
   - Command name: lowercase, kebab-case only
   - Purpose: non-empty string

2. **Generate file content** using this template:

```markdown
---
description: { { PURPOSE } }
argument-hint: [args...]
---

# /{{COMMAND_NAME}}

## Purpose

{{PURPOSE}}

## Contract

**Inputs:** `$ARGUMENTS` — command arguments
**Outputs:** `STATUS=<OK|FAIL> [key=value ...]`

## Instructions

1. **Validate inputs:**
   - Check that required arguments are provided
   - Validate argument format/values

2. **Execute the command:**

\`\`\`bash

# Add specific bash commands here

# Example: git worktree add path/to/worktree branch-name

\`\`\`

3. **Output status:**
   - Print `STATUS=OK` on success
   - Print `STATUS=FAIL ERROR="message"` on failure

## Constraints

- Idempotent and deterministic
- No network tools by default
- Minimal console output (STATUS line only)
```

3. **Output:**
   - Show preview in fenced markdown block
   - Create `.claude/commands/` dir and write file atomically
   - Print: `STATUS=WROTE PATH=.claude/commands/{{COMMAND_NAME}}.md`

## Example

```bash
/create-command analyze-deps "Analyze dependencies for outdated packages"
# Output: STATUS=WROTE PATH=.claude/commands/analyze-deps.md
```
