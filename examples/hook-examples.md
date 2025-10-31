# Hook Examples

Practical examples of hooks for common automation scenarios.

## Code formatting hooks

### TypeScript/JavaScript formatting

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

### Python formatting with Black

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.py$'; then black \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

### Go formatting

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.go$'; then gofmt -w \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

## Logging and auditing hooks

### Bash command logger

Log all bash commands that Claude runs:

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

### File modification tracker

Track all file modifications:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"[\\(now | strftime(\"%Y-%m-%d %H:%M:%S\"))] Modified: \\(.tool_input.file_path)\"' >> ~/.claude/file-changes.log"
          }
        ]
      }
    ]
  }
}
```

## Notification hooks

### Desktop notification (macOS)

```json
{
  "hooks": {
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude Code is waiting for your input\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### Desktop notification (Linux)

```json
{
  "hooks": {
    "Notification": [
      {
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

## Security and validation hooks

### File protection hook

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

### Command validation hook

Validate bash commands before execution:

```bash
#!/usr/bin/env python3
import json
import sys

data = json.load(sys.stdin)
command = data.get('tool_input', {}).get('command', '')

# Block dangerous commands
dangerous_patterns = ['rm -rf /', 'dd if=', 'mkfs', ':(){ :|:& };:']

for pattern in dangerous_patterns:
    if pattern in command:
        print(f"Blocked dangerous command containing: {pattern}", file=sys.stderr)
        sys.exit(2)  # Block the command

sys.exit(0)  # Allow the command
```

Configuration:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/validate-command.py"
          }
        ]
      }
    ]
  }
}
```

## Session management hooks

### SessionStart: Environment setup

```bash
#!/bin/bash

# Load project-specific environment
if [ -f ".env.local" ]; then
    if [ -n "$CLAUDE_ENV_FILE" ]; then
        # Parse .env.local and add to CLAUDE_ENV_FILE
        while IFS='=' read -r key value; do
            # Skip comments and empty lines
            [[ $key =~ ^#.*$ ]] && continue
            [[ -z $key ]] && continue
            echo "export $key='$value'" >> "$CLAUDE_ENV_FILE"
        done < .env.local
    fi
fi

# Output project context
echo "## Project Context"
echo "Current branch: $(git branch --show-current)"
echo "Last commit: $(git log -1 --oneline)"
echo "Uncommitted changes: $(git status --short | wc -l)"

exit 0
```

### SessionEnd: Cleanup

```bash
#!/bin/bash

# Log session end
echo "[$(date)] Session ended: $session_id" >> ~/.claude/session-history.log

# Archive conversation if successful
if [ "$permission_mode" != "error" ]; then
    cp "$transcript_path" ~/.claude/archives/
fi

exit 0
```

## Testing hooks

### Auto-run tests after code changes

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q 'src/.*\\.ts$'; then npm test; fi; }",
            "timeout": 60
          }
        ]
      }
    ]
  }
}
```

## Related documentation

For more information on creating and using hooks, see [hooks.md](../hooks.md).
