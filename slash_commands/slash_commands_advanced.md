# How to Speed Up Your Claude Code Experience with Slash Commands

import Alert from "@components/Alert.astro";
import Aside from "@components/Aside.astro";
import PromptLink from "@components/PromptLink.astro";
import FileTree from "@components/FileTree.astro";
import Figure from "@components/Figure.astro";
import InternalLink from "@components/InternalLink.astro";
import promptSlash from "@assets/images/claude/promptSlash.png";
import terminalTest from "@assets/images/claude/terminalTest.png";

I was wasting time. Every commit message, every branch name, every PR description. I typed the same things over and over.
Then I discovered Slash Commands in Claude Code. Now I type `/commit` and it writes the message for me. `/branch "add dark mode"` and it creates `feat/add-dark-mode`. `/pr` and it generates a full PR description from my commits.
This post shows you how to build the same workflow. I'll cover how Slash Commands work, then we'll build a complete system that automates your entire git lifecycle.

<Aside type="tip" title="Related Reading">
Slash Commands are just one piece of the Claude Code puzzle. For the full picture—including MCP, hooks, subagents, and skills—see my <InternalLink slug="understanding-claude-code-full-stack">comprehensive guide to Claude Code's feature stack</InternalLink>.
</Aside>

<Figure src={terminalTest} alt="Claude Code executing /branch command showing git operations" caption="The /branch command creates a feature branch automatically" />

<Alert type="important" title="Prerequisites">
You need Git and the GitHub CLI (`gh`). Install `gh` with `brew install gh` on macOS or check [cli.github.com](https://cli.github.com). Run `gh auth login` to authenticate.

Without `gh`, commands like `/pr` and `/fix-pipeline` will not work.
</Alert>


## Two things you need to know

Before we build the workflow, you need to understand two features.

### Bash command execution

Write `!git status` inside a command file. Claude runs the command first, captures the output, and injects it into the prompt. The AI sees the result before it starts thinking.

This is how `/commit` knows what you changed. It runs `!git diff` automatically. See the [official documentation](https://docs.anthropic.com/en/docs/claude-code/slash-commands#bash-command-execution) for more details.

### Model selection

You don't need a powerful model to fix a missing semicolon.
Claude Code lets you pick the model in the frontmatter:

- `sonnet` — for complex reasoning (default)
- `haiku` — fast and cheap

Add `model: haiku` and commands run almost instantly.

## Command structure

Slash commands are Markdown files stored in `.claude/commands/` (project-level) or `~/.claude/commands/` (personal). The filename becomes the command name: `commit.md` becomes `/commit`.

<FileTree
  tree={[
    {
      name: "my-project",
      open: true,
      children: [
        {
          name: ".claude",
          open: true,
          children: [
            { name: "settings.json", comment: "project settings" },
            {
              name: "commands",
              open: true,
              children: [
                { name: "branch.md", comment: "/branch" },
                { name: "commit.md", comment: "/commit" },
                { name: "pr.md", comment: "/pr" },
                { name: "lint.md", comment: "/lint" },
              ],
            },
          ],
        },
        { name: "src" },
        { name: "package.json" },
      ],
    },
  ]}
/>

Here is a complete example:

```markdown
---
description: Create a git commit with a conventional message
allowed-tools: Bash(git add:*), Bash(git commit:*)
argument-hint: [message]
model: haiku
---

# Commit Changes

<git_diff>
!`git diff --cached`
</git_diff>

Create a commit message following Conventional Commits.
If $ARGUMENTS is provided, use it as the commit message.
```

<Figure src={promptSlash} alt="Claude Code slash command autocomplete showing available commands" caption="Slash commands appear in autocomplete when you type /" />

### Frontmatter options

| Option | Purpose | Default |
|--------|---------|---------|
| `description` | Brief description shown in `/help` | First line of prompt |
| `allowed-tools` | Tools the command can use | Inherits from conversation |
| `model` | Model to use (`sonnet`, `haiku`, or full model ID) | Inherits from conversation |
| `argument-hint` | Shows expected arguments in autocomplete | None |

### Arguments

Use `$ARGUMENTS` to capture everything passed to the command:

```markdown
Create a branch named: $ARGUMENTS
```

For multiple arguments, use positional parameters `$1`, `$2`, etc:

```markdown
---
argument-hint: [pr-number] [priority]
---

Review PR #$1 with priority $2.
```

### File references

Include file contents with the `@` prefix:

```markdown
Review the implementation in @src/utils/helpers.js
```

## The workflow

I replaced my manual git rituals with custom commands. They live in `.claude/commands/`. Here is how I drive a feature from start to merge.

```mermaid
---
title: Development Workflow
---
flowchart LR
    %% Initial Setup
    Start((Start)) --> Branch["/branch"]
    Branch --> Code[Write Code]

    %% Local Iteration Loop
    Code --> Lint["/lint<br/>(Haiku)"]
    Lint -- "Auto-fix" --> Lint
    Lint --> Test["/vitest<br/>(Haiku)"]
    Test -- "Fix Failure" --> Test
    
    %% Deployment Flow
    Test --> Push["/push"]
    Push --> PR["/pr"]
    PR --> CI{CI Pass?}

    %% CI Debugging Loop
    CI -- "No" --> Fix["/fix-pipeline<br/>(Sonnet)"]
    Fix -- "Fix & Push" --> CI

    %% Final Review & Merge
    CI -- "Yes" --> Review["/review-coderabbit"]
    Review --> Merge["/merge-to-main"]
    Merge --> Done((Done))
```

### /branch — start a task

<PromptLink href="/prompts/claude/claude-branch-command">
I type `/branch "implement dark mode toggle"` and Claude checks out main, pulls latest, and creates `feat/dark-mode-toggle`. No more thinking about naming conventions.
</PromptLink>

### /lint — fix before commit

<PromptLink href="/prompts/claude/claude-lint-command">
I type `/lint`. It runs the linter with auto-fix, and if errors remain, Claude fixes them. Uses Haiku for speed—runs in about 20 seconds.
</PromptLink>

### /vitest — run unit tests

<PromptLink href="/prompts/claude/claude-vitest-command">
I type `/vitest`. It runs the test suite and fixes any failures. The prompt tells Claude to fix the code, not the test—implementation should match expected behavior.
</PromptLink>

### /commit — save your work

<PromptLink href="/prompts/claude/claude-commit-command">
I type `/commit`. Claude analyzes the diff, generates a Conventional Commit message, and commits. It looks at recent commits to match your project's style.
</PromptLink>

### /push — commit and push in one step

<PromptLink href="/prompts/claude/claude-push-command">
I type `/push`. It stages everything, generates a commit message, commits, and pushes. My most-used command—one word and the code is on GitHub.
</PromptLink>

### /fix-pipeline — fix failing CI tests

<PromptLink href="/prompts/claude/claude-fix-pipeline-command">
I type `/fix-pipeline`. It fetches the failed logs via `gh`, analyzes the error, and fixes it. Uses Sonnet because debugging requires reasoning. The prompt includes guardrails—Claude must read the actual error before proposing fixes.
</PromptLink>

### /pr — create a pull request

<PromptLink href="/prompts/claude/claude-pr-command">
I type `/pr`. It analyzes all commits on the branch, generates a PR title and description, and opens it via `gh pr create`. Checks if a PR already exists first.
</PromptLink>

### /review-coderabbit — address review comments

<PromptLink href="/prompts/claude/claude-review-coderabbit-command">
I type `/review-coderabbit`. It fetches CodeRabbit's comments via GraphQL, verifies each suggestion against the codebase, implements valid fixes or pushes back with reasoning, and resolves every thread. AI reviewers aren't always right—the prompt ensures Claude verifies before acting.
</PromptLink>

### /merge-to-main — finish the task

<PromptLink href="/prompts/claude/claude-merge-to-main-command">
I type `/merge-to-main`. It squash merges the PR, deletes the branch, and pulls main. Done.
</PromptLink>

## Summary

By moving your process into `.claude/commands/`, you are building a system.

- Bash command execution injects real-time context
- Model selection balances speed vs reasoning
- The workflow automates branching, linting, committing, CI debugging, PRs, and merging

Define the process once. Claude executes it every time.

Want to extend Claude Code even further? Connect external tools via <InternalLink slug="what-is-model-context-protocol-mcp">MCP (Model Context Protocol)</InternalLink> or package your commands into a <InternalLink slug="building-my-first-claude-code-plugin">shareable plugin</InternalLink>.

I don't think about naming conventions, commit messages, or PR descriptions anymore. The commands handle it.

<Alert type="tip" title="Bonus: Shell aliases for even faster execution">
You can skip the interactive prompt entirely with `claude -p`. Add aliases to your `.zshrc` or `.bashrc`:

```bash
alias clint="claude -p '/lint'"
alias cpush="claude -p '/push'"
alias ccommit="claude -p '/commit'"
alias cbranch="claude -p '/branch'"
```

Now `clint` runs the lint command without opening the interactive session. The `-p` flag passes the prompt directly—Claude executes and exits. Two steps become one keystroke.
</Alert>
