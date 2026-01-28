# Get Notified When Claude Code Finishes With Hooks

import FileTree from '@components/FileTree.astro'
import InternalLink from '@components/InternalLink.astro'

You're deep in your work. Claude Code is running, doing its thing. You check back five minutes later. Still waiting. Ten minutes later? Still waiting.

Wouldn't it be nice to know *when* Claude actually needs you?

This is where hooks come in. <InternalLink slug="understanding-claude-code-full-stack">Claude Code runs hooks</InternalLink> at specific points in its workflow. You can tap into those hooks to send yourself desktop notifications—so you never miss an important moment.

But here's the thing—if you've never used hooks before, they might sound abstract. Let me break it down.

## What Are Hooks?

Hooks are commands that run at specific points in Claude Code's lifecycle. They let you respond to events without constantly watching the CLI. Instead of polling, you get notified.

Claude Code provides two notification hooks:
- **`permission_prompt`** - Claude needs your permission to do something
- **`idle_prompt`** - Claude is waiting for your input

Think of them like webhooks, but for your local machine. Claude Code fires an event, you can respond.

## Setting Up Desktop Notifications

Now let's get this working. It's straightforward—just two pieces: a configuration file and a notification script.

Start by creating a `.claude/hooks` directory in your project. Then add the hook configuration to `.claude/settings.json`:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt|idle_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "npx tsx \"$CLAUDE_PROJECT_DIR/.claude/hooks/notification-desktop.ts\"",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

This tells Claude Code: "When you hit a `permission_prompt` or `idle_prompt`, run this command." The `timeout: 5` means the hook has 5 seconds to complete before Claude moves on.

You can place this in two locations:
- `.claude/settings.json` - Project-specific (checked into git, shared with team)
- `~/.claude/settings.json` - Global user settings (personal machine only)

Use project-specific settings for team hooks, global settings for personal notifications. The `$CLAUDE_PROJECT_DIR` is an environment variable Claude Code provides—it expands to your project root automatically.

Here's what your project structure should look like:

<FileTree tree={[
  { name: 'your-project', children: [
    { name: '.claude', children: [
      { name: 'settings.json', comment: 'Hook configuration' },
      { name: 'hooks', children: [
        { name: 'notification-desktop.ts', comment: 'Notification script' }
      ], open: true }
    ], open: true },
    { name: 'src', comment: '...' },
    { name: 'package.json', comment: '...' }
  ], open: true }
]} />

## The Notification Script

Create `.claude/hooks/notification-desktop.ts`. This script handles sending the actual notifications:

```typescript
#!/usr/bin/env npx tsx
/* eslint-disable node/prefer-global/process */
/**
 * Claude Code Notification Hook - Desktop Alerts
 *
 * Sends system notifications when Claude needs attention:
 * - Permission prompts
 * - Idle prompts (waiting for input)
 */

import type { NotificationHookInput } from '@anthropic-ai/claude-agent-sdk'

import { execSync } from 'node:child_process'
import { readFileSync } from 'node:fs'

function readStdin(): string {
  return readFileSync(0, 'utf-8')
}

function sendMacNotification(title: string, message: string): void {
  // Escape special characters for AppleScript
  const escapedTitle = title.replace(/"/g, '\\"')
  const escapedMessage = message.replace(/"/g, '\\"')

  const script = `display notification "${escapedMessage}" with title "${escapedTitle}" sound name "Ping"`

  try {
    execSync(`osascript -e '${script}'`, { stdio: 'ignore' })
  }
  catch {
    // Notification failed, ignore silently
  }
}

function main(): void {
  const rawInput = readStdin()

  let parsedInput: unknown
  try {
    parsedInput = JSON.parse(rawInput)
  }
  catch {
    process.exit(0)
  }

  const input = parsedInput as NotificationHookInput

  const notificationType = (input as { notification_type?: string }).notification_type
  const message = input.message

  switch (notificationType) {
    case 'permission_prompt':
      sendMacNotification('Claude Code - Permission Required', message || 'Claude needs your permission to continue')
      break
    case 'idle_prompt':
      sendMacNotification('Claude Code - Waiting', message || 'Claude is waiting for your input')
      break
    default:
      // Don't notify for other types
      break
  }

  process.exit(0)
}

main()

```

## When This Really Shines

You've set up the basics. Now here's where it becomes powerful.

This notification system is especially useful when you're doing deep focus work and Claude Code runs a long operation. You don't have to check your terminal every few seconds. Permission prompts that need immediate action? They hit you with a different sound. Idle waits while you've stepped away? A gentle reminder pulls you back.

The key is this: the notification comes exactly when you need to be engaged. No sooner, no later.

Want to go deeper? You can even <InternalLink slug="building-my-first-claude-code-plugin">build custom plugins that use hooks</InternalLink> across different projects for more powerful automation.

## Conclusion

Hooks transform Claude Code from a tool you watch into a tool that watches for you.
The setup takes maybe five minutes. Copy the configuration, create the script, adjust the sounds to your preference. After that, you're done. No more context switching. No more glancing at the terminal every few seconds wondering if Claude needs you. The notification arrives exactly when it matters.
That's the real power here. It's not about automating notifications. It's about reclaiming your focus—letting Claude Code work while you work, and pulling your attention back only when it's needed. Set it up once, and you've unlocked a better way to collaborate with AI.
