# Claude Code Driver

**Actionable context for building Claude Code extensions, skills, and plugins.**

This repository serves as a **Knowledge Base** (or "Driver") for Large Language Models. It aggregates expert tutorials, best practices, and documentation patterns for extending [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview).

By feeding this repository into an LLM session (using tools like [gitingest](https://gitingest.com/)), you teach the model exactly how to build high-quality Skills, Subagents, Hooks, and Plugins that follow the latest standards.

## 🚀 How to Use

### 1. Ingest the Context
Use `gitingest` to turn this repository into a single text prompt.
- Go to [gitingest.com](https://gitingest.com/)
- Enter this repository URL.
- Copy the resulting text.

### 2. Feed the LLM
Paste the ingested text into your Claude Code session, ChatGPT, or Claude.ai web interface.

### 3. Trigger the "Extension Developer" Persona
Once the context is loaded, use the following system prompt to start building:

```markdown
I have provided you with the "Claude Code Driver" context. You are now an expert in the Claude Code architecture.

Your goal is to help me build a specific extension component. 
Please reference the patterns found in `main.md` and the specific sub-directories (skills, hooks, etc.) to ensure best practices.

My request is: [INSERT YOUR GOAL HERE]

Examples of goals:
- "Create a Skill that automatically detects when I'm writing SQL and offers to optimize it."
- "Build a Plugin for my team that enforces our TDD workflow."
- "Write a Hook that plays a sound when a long-running task finishes."
- "Create a Slash Command that scaffolds a new Vue.js component."
```

## 📂 Repository Structure

This repository is organized by feature type. Each directory contains actionable guides and templates.

| File/Directory | Description |
| :--- | :--- |
| **`main.md`** | **Start here.** The "Theory of Everything" for Claude Code. Explains how MCP, Skills, Subagents, and Hooks stack together. |
| **`skills/`** | Documentation on **Agent Skills**—capabilities that trigger *automatically* based on context (e.g., specific file types or tasks). |
| **`subagents/`** | How to build specialized AI personalities (e.g., a "Security Auditor" or "QA Engineer") that run in isolated contexts. |
| **`slash_commands/`** | Guides for creating manual `/commands` (e.g., `/commit`, `/pr`) to automate repetitive workflows. |
| **`hooks/`** | Configuration for event-driven actions (e.g., running linting after a file edit or sending desktop notifications). |
| **`plugins/`** | How to bundle all the above into shareable `.claude-plugin` packages for your team or the marketplace. |
| **`claude_md/`** | Best practices for `CLAUDE.md` files—project memory and conventions. |

## 🧠 What Can I Build?

Using this context, you can instruct Claude to build:

1.  **Context-Aware Skills:** "Create a skill that reads my `schema.prisma` whenever I ask about database queries."
2.  **Automated Workflows:** "Write a hook that runs `npm test` every time I edit a `.test.ts` file."
3.  **Team Standards:** "Package our linting rules and commit message format into a Plugin I can share with the junior devs."
4.  **Specialized Agents:** "Create a subagent specifically for auditing smart contracts."

## 📚 Credits & Attribution

The core content of this repository is curated from the excellent tutorials and writings of **Alexander Opalic**.

- **Source:** [Alexander Opalic on GitHub](https://github.com/alexanderop)
- **Blog:** Check `main.md` for links to original articles.

This repository aggregates these resources into a format optimized for LLM consumption ("Context Stuffing").

## 📄 License

MIT License. See [LICENSE](LICENSE) file for details.
Copyright (c) 2026 thewiredbear.
