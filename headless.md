# Headless mode

> Run Claude Code programmatically without interactive UI

## Overview

Headless mode allows you to run Claude Code programmatically from command line scripts and automation tools without any interactive UI.

## Basic usage

Use the `--print` (or `-p`) flag to run in non-interactive mode and print the final result:

```bash
claude -p "Stage my changes and write a set of commits for them" \
  --allowedTools "Bash,Read" \
  --permission-mode acceptEdits
```

## Configuration Options

Key flags for automation and scripting:

| Flag                       | Description                                                     | Example                                  |
| :------------------------- | :-------------------------------------------------------------- | :--------------------------------------- |
| `--print`, `-p`            | Run in non-interactive mode                                     | `claude -p "query"`                      |
| `--output-format`          | Specify output format (`text`, `json`, `stream-json`)           | `claude -p --output-format json`         |
| `--resume`, `-r`           | Resume a conversation by session ID                             | `claude --resume abc123`                 |
| `--continue`, `-c`         | Continue the most recent conversation                           | `claude --continue`                      |
| `--verbose`                | Enable verbose logging                                          | `claude --verbose`                       |
| `--append-system-prompt`   | Append to system prompt (only with `--print`)                   | `claude --append-system-prompt "Custom"` |
| `--allowedTools`           | Comma-separated list of allowed tools                           | `claude --allowedTools "Bash,Read"`      |
| `--disallowedTools`        | Comma-separated list of denied tools                            | `claude --disallowedTools "WebFetch"`    |
| `--mcp-config`             | Load MCP servers from a JSON file                               | `claude --mcp-config servers.json`       |
| `--permission-prompt-tool` | MCP tool for handling permission prompts (only with `--print`)  | `claude --permission-prompt-tool mcp__auth` |

For a complete list of CLI options, see [cli-reference.md](cli-reference.md).

## Multi-turn conversations

For multi-turn conversations, resume conversations or continue from the most recent session:

```bash
# Continue the most recent conversation
claude --continue "Now refactor this for better performance"

# Resume a specific conversation by session ID
claude --resume 550e8400-e29b-41d4-a716-446655440000 "Update the tests"

# Resume in non-interactive mode
claude --resume 550e8400-e29b-41d4-a716-446655440000 "Fix all linting issues" --no-interactive
```

## Output Formats

### Text Output (Default)

```bash
claude -p "Explain file src/components/Header.tsx"
# Output: This is a React component showing...
```

### JSON Output

Returns structured data including metadata:

```bash
claude -p "How does the data layer work?" --output-format json
```

Response format:

```json
{
  "type": "result",
  "subtype": "success",
  "total_cost_usd": 0.003,
  "is_error": false,
  "duration_ms": 1234,
  "duration_api_ms": 800,
  "num_turns": 6,
  "result": "The response text here...",
  "session_id": "abc123"
}
```

### Streaming JSON Output

Streams each message as it is received:

```bash
claude -p "Build an application" --output-format stream-json
```

Each conversation begins with an initial `init` system message, followed by user and assistant messages, followed by a final `result` system message with stats.

## Input Formats

### Text Input (Default)

```bash
# Direct argument
claude -p "Explain this code"

# From stdin
echo "Explain this code" | claude -p
```

### Streaming JSON Input

A stream of messages provided via `stdin` where each message represents a user turn. Requires `-p` and `--output-format stream-json`:

```bash
echo '{"type":"user","message":{"role":"user","content":[{"type":"text","text":"Explain this code"}]}}' | claude -p --output-format=stream-json --input-format=stream-json
```

## Agent Integration Examples

### SRE Incident Response Bot

```bash
#!/bin/bash

# Automated incident response agent
investigate_incident() {
    local incident_description="$1"
    local severity="${2:-medium}"

    claude -p "Incident: $incident_description (Severity: $severity)" \
      --append-system-prompt "You are an SRE expert. Diagnose the issue, assess impact, and provide immediate action items." \
      --output-format json \
      --allowedTools "Bash,Read,WebSearch,mcp__datadog" \
      --mcp-config monitoring-tools.json
}

# Usage
investigate_incident "Payment API returning 500 errors" "high"
```

### Automated Security Review

```bash
# Security audit agent for pull requests
audit_pr() {
    local pr_number="$1"

    gh pr diff "$pr_number" | claude -p \
      --append-system-prompt "You are a security engineer. Review this PR for vulnerabilities, insecure patterns, and compliance issues." \
      --output-format json \
      --allowedTools "Read,Grep,WebSearch"
}

# Usage and save to file
audit_pr 123 > security-report.json
```

### Multi-turn Legal Assistant

```bash
# Legal document review with session persistence
session_id=$(claude -p "Start legal review session" --output-format json | jq -r '.session_id')

# Review contract in multiple steps
claude -p --resume "$session_id" "Review contract.pdf for liability clauses"
claude -p --resume "$session_id" "Check compliance with GDPR requirements"
claude -p --resume "$session_id" "Generate executive summary of risks"
```

## Best practices

* **Use JSON output format** for programmatic parsing:
  ```bash
  result=$(claude -p "Generate code" --output-format json)
  code=$(echo "$result" | jq -r '.result')
  cost=$(echo "$result" | jq -r '.cost_usd')
  ```

* **Handle errors gracefully** - check exit codes and stderr:
  ```bash
  if ! claude -p "$prompt" 2>error.log; then
      echo "Error occurred:" >&2
      cat error.log >&2
      exit 1
  fi
  ```

* **Use session management** for maintaining context in multi-turn conversations

* **Consider timeouts** for long-running operations:
  ```bash
  timeout 300 claude -p "$complex_prompt" || echo "Timed out after 5 minutes"
  ```

* **Respect rate limits** when making multiple requests by adding delays between calls

## Related documentation

* [cli-reference.md](cli-reference.md) - Complete CLI documentation
* [configuration.md](configuration.md) - Settings and configuration
* [permissions.md](permissions.md) - Permission modes
