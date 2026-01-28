Slash command to create custom subagents for specialized tasks with proper configuration and system prompts

💡 Use Case: Use when building custom Claude Code agents for code review, debugging, or specialized development tasks

---

description: Create a new custom subagent for specialized tasks
argument-hint: [name] [description]

---

## /create-agent

## Purpose

Create a new custom subagent (skill) for specialized tasks with proper configuration and system prompts.

## Contract

**Inputs:**

- `$1` — AGENT_NAME (lowercase, kebab-case, e.g., "code-reviewer")
- `$2` — DESCRIPTION (when this agent should be invoked, e.g., "Expert code reviewer. Use proactively after code changes.")
- `--user` — Create user-level agent in `~/.claude/agents/` (default: project-level in `.claude/agents/`)
- `--tools` — Comma-separated list of tools (e.g., "Read,Grep,Glob,Bash")
- `--model` — Model to use: "sonnet", "opus", "haiku", or "inherit" (default: sonnet)

**Outputs:**

- `STATUS=<CREATED|EXISTS|FAIL> PATH=<path> AGENT=<name>`

## Instructions

1. **Validate inputs:**
   - Agent name must be lowercase, kebab-case only
   - Description must be non-empty and descriptive
   - If `--tools` specified, validate tool names against available tools
   - If `--model` specified, validate it's one of: sonnet, opus, haiku, inherit

2. **Determine file location:**
   - Default: `.claude/agents/{AGENT_NAME}.md` (project-level)
   - With `--user`: `~/.claude/agents/{AGENT_NAME}.md` (user-level)
   - Create directory if it doesn't exist

3. **Check for existing agent:**
   - If file exists, output `STATUS=EXISTS` and exit
   - Recommend using `/agents` command to edit existing agents

4. **Generate agent file content:**

   ```markdown
   ---
   name: { AGENT_NAME }
   description: { DESCRIPTION }
   tools: { TOOLS } # Optional - only include if specified
   model: { MODEL } # Optional - only include if specified
   ---

   You are a specialized agent for {purpose based on description}.

   When invoked:

   1. Understand the specific task or problem
   2. Analyze the relevant context
   3. Execute your specialized function
   4. Provide clear, actionable results

   Key responsibilities:

   - {Responsibility 1 based on description}
   - {Responsibility 2 based on description}
   - {Responsibility 3 based on description}

   Best practices:

   - Be focused and efficient
   - Provide specific, actionable feedback
   - Document your reasoning
   - Follow established patterns and conventions

   For each task:

   - Explain your approach
   - Show your work
   - Highlight key findings or changes
   - Suggest next steps if applicable
   ```

5. **Write the file:**
   - Create the agent file with proper frontmatter
   - Ensure proper formatting and indentation
   - Set appropriate file permissions

6. **Output result:**
   - Print: `STATUS=CREATED PATH={path} AGENT={name}`
   - Provide usage example: `Use the {name} agent to...`

## Examples

### Basic project-level agent

```bash
/create-skill code-reviewer "Expert code review specialist. Use proactively after code changes."
# Output: STATUS=CREATED PATH=.claude/agents/code-reviewer.md AGENT=code-reviewer
```

### User-level agent with specific tools

```bash
/create-skill debugger "Debugging specialist for errors and test failures" --user --tools "Read,Edit,Bash,Grep,Glob"
# Output: STATUS=CREATED PATH=~/.claude/agents/debugger.md AGENT=debugger
```

### Agent with specific model

```bash
/create-skill data-scientist "Data analysis expert for SQL queries and insights" --tools "Bash,Read,Write" --model "sonnet"
# Output: STATUS=CREATED PATH=.claude/agents/data-scientist.md AGENT=data-scientist
```

## Constraints

- Idempotent: Won't overwrite existing agents
- No network access required
- Agent names must follow naming conventions
- Tools list must match available Claude Code tools
- Model must be valid alias or 'inherit'

## Best Practices

- Use descriptive, action-oriented descriptions
- Include "use PROACTIVELY" in description for automatic invocation
- Start with Claude-generated agents via `/agents` for complex cases
- Limit tools to only what the agent needs
- Design focused agents with single, clear responsibilities
- Add to version control for team collaboration

## Related Commands

- `/agents` - Interactive interface for managing agents (recommended for editing)
- Use created agents: "Use the {name} agent to..."
- Invoke explicitly: "Ask the {name} agent to investigate..."
