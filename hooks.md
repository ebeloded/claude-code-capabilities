# Get started with Claude Code hooks

> Learn how to customize and extend Claude Code's behavior by registering shell commands

Claude Code hooks are user-defined shell commands that execute at various points in Claude Code's lifecycle. Hooks provide deterministic control over Claude Code's behavior, ensuring certain actions always happen rather than relying on the LLM to choose to run them.

## Example use cases

* **Notifications**: Customize how you get notified when Claude Code is awaiting your input or permission to run something.
* **Automatic formatting**: Run `prettier` on .ts files, `gofmt` on .go files, etc. after every file edit.
* **Logging**: Track and count all executed commands for compliance or debugging.
* **Feedback**: Provide automated feedback when Claude Code produces code that does not follow your codebase conventions.
* **Custom permissions**: Block modifications to production files or sensitive directories.

By encoding these rules as hooks rather than prompting instructions, you turn suggestions into app-level code that executes every time it is expected to run.

**Warning**: You must consider the security implication of hooks as you add them, because hooks run automatically during the agent loop with your current environment's credentials. For example, malicious hooks code can exfiltrate your data. Always review your hooks implementation before registering them.

## Hook Events Overview

Claude Code provides several hook events that run at different points in the workflow:

* **PreToolUse**: Runs before tool calls (can block them)
* **PostToolUse**: Runs after tool calls complete
* **UserPromptSubmit**: Runs when the user submits a prompt, before Claude processes it
* **Notification**: Runs when Claude Code sends notifications
* **Stop**: Runs when Claude Code finishes responding
* **SubagentStop**: Runs when subagent tasks complete
* **PreCompact**: Runs before Claude Code is about to run a compact operation
* **SessionStart**: Runs when Claude Code starts a new session or resumes an existing session
* **SessionEnd**: Runs when Claude Code session ends

Each event receives different data and can control Claude's behavior in different ways.

## Quickstart

In this quickstart, you'll add a hook that logs the shell commands that Claude Code runs.

### Prerequisites

Install `jq` for JSON processing in the command line.

### Step 1: Open hooks configuration

Run the `/hooks` slash command and select the `PreToolUse` hook event.

`PreToolUse` hooks run before tool calls and can block them while providing Claude feedback on what to do differently.

### Step 2: Add a matcher

Select `+ Add new matcher…` to run your hook only on Bash tool calls.

Type `Bash` for the matcher.

**Note:** You can use `*` to match all tools.

### Step 3: Add the hook

Select `+ Add new hook…` and enter this command:

```bash
jq -r '"\(.tool_input.command) - \(.tool_input.description // "No description")"' >> ~/.claude/bash-command-log.txt
```

### Step 4: Save your configuration

For storage location, select `User settings` since you're logging to your home directory. This hook will then apply to all projects, not just your current project.

Then press Esc until you return to the REPL. Your hook is now registered!

### Step 5: Verify your hook

Run `/hooks` again or check `~/.claude/settings.json` to see your configuration:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"\\(.tool_input.command) - \\(.tool_input.description // \"No description\")\"' >> ~/.claude/bash-command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### Step 6: Test your hook

Ask Claude to run a simple command like `ls` and check your log file:

```bash
cat ~/.claude/bash-command-log.txt
```

You should see entries like:

```
ls - Lists files and directories
```

## More Examples

### Code Formatting Hook

Automatically format TypeScript files after editing:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.ts$'; then npx prettier --write \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

### Custom Notification Hook

Get desktop notifications when Claude needs input:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Awaiting your input'"
          }
        ]
      }
    ]
  }
}
```

### File Protection Hook

Block edits to sensitive files:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); sys.exit(2 if any(p in path for p in ['.env', 'package-lock.json', '.git/']) else 0)\""
          }
        ]
      }
    ]
  }
}
```

## Configuration

Hooks are configured in your settings files:

* `~/.claude/settings.json` - User settings
* `.claude/settings.json` - Project settings
* `.claude/settings.local.json` - Local project settings (not committed)
* Enterprise managed policy settings

### Structure

```json
{
  "hooks": {
    "EventName": [
      {
        "matcher": "ToolPattern",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

* **matcher**: Pattern to match tool names, case-sensitive
  * Simple strings match exactly: `Write` matches only the Write tool
  * Supports regex: `Edit|Write` or `Notebook.*`
  * Use `*` to match all tools
* **hooks**: Array of commands to execute when the pattern matches
  * `type`: Currently only `"command"` is supported
  * `command`: The bash command to execute
  * `timeout`: (Optional) How long a command should run, in seconds

### Project-Specific Hook Scripts

You can use the environment variable `CLAUDE_PROJECT_DIR` to reference scripts stored in your project:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/check-style.sh"
          }
        ]
      }
    ]
  }
}
```

### Plugin hooks

Plugins can provide hooks that integrate seamlessly with your user and project hooks. Plugin hooks are automatically merged with your configuration when plugins are enabled.

Plugin hooks use the `${CLAUDE_PLUGIN_ROOT}` environment variable to reference plugin files.

See [plugins.md](plugins.md) for details on creating plugin hooks.

## Hook Input

Hooks receive JSON data via stdin containing session information and event-specific data:

```typescript
{
  // Common fields
  session_id: string
  transcript_path: string  // Path to conversation JSON
  cwd: string              // Current working directory
  permission_mode: string  // Current permission mode

  // Event-specific fields
  hook_event_name: string
  ...
}
```

## Hook Exit Codes

Hooks communicate results through exit codes:

* **0**: Success - proceed normally
* **1**: Soft failure - log error, proceed anyway
* **2**: Hard failure - block operation (PreToolUse only)
* **3**: Cancel - stop current operation

For PreToolUse hooks:
* Exit code 2 blocks the tool call and shows hook output to Claude
* Other exit codes allow the tool call to proceed

## SessionStart Hooks

SessionStart hooks can persist environment variables using the `CLAUDE_ENV_FILE` environment variable:

```bash
#!/bin/bash

if [ -n "$CLAUDE_ENV_FILE" ]; then
  echo 'export NODE_ENV=production' >> "$CLAUDE_ENV_FILE"
  echo 'export API_KEY=your-api-key' >> "$CLAUDE_ENV_FILE"
fi

exit 0
```

Any variables written to this file will be available in all subsequent bash commands that Claude Code executes during the session.

## Related documentation

* [permissions.md](permissions.md) - Fine-grained access control
* [configuration.md](configuration.md) - Settings files and configuration
* [plugins.md](plugins.md) - Create plugin hooks
