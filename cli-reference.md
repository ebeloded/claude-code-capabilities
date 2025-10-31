# CLI reference

> Complete reference for Claude Code command-line interface, including commands and flags.

## CLI commands

| Command                            | Description                                    | Example                               |
| :--------------------------------- | :--------------------------------------------- | :------------------------------------ |
| `claude`                           | Start interactive REPL                         | `claude`                              |
| `claude "query"`                   | Start REPL with initial prompt                 | `claude "explain this project"`       |
| `claude -p "query"`                | Query via SDK, then exit                       | `claude -p "explain this function"`   |
| `cat file \| claude -p "query"`    | Process piped content                          | `cat logs.txt \| claude -p "explain"` |
| `claude -c`                        | Continue most recent conversation              | `claude -c`                           |
| `claude -c -p "query"`             | Continue via SDK                               | `claude -c -p "Check for errors"`     |
| `claude -r "<session-id>" "query"` | Resume session by ID                           | `claude -r "abc123" "Finish this"`    |
| `claude update`                    | Update to latest version                       | `claude update`                       |
| `claude mcp`                       | Configure Model Context Protocol (MCP) servers | `claude mcp`                          |

## CLI flags

| Flag                             | Description                                                     | Example                                                                                            |
| :------------------------------- | :-------------------------------------------------------------- | :------------------------------------------------------------------------------------------------- |
| `--add-dir`                      | Add additional working directories                              | `claude --add-dir ../apps ../lib`                                                                  |
| `--agents`                       | Define custom subagents dynamically via JSON                    | `claude --agents '{"reviewer":{"description":"Reviews code","prompt":"You are a code reviewer"}}'` |
| `--allowedTools`                 | Tools that should be allowed without prompting                  | `"Bash(git log:*)" "Bash(git diff:*)" "Read"`                                                      |
| `--disallowedTools`              | Tools that should be disallowed                                 | `"Bash(git log:*)" "Bash(git diff:*)" "Edit"`                                                      |
| `--print`, `-p`                  | Print response without interactive mode                         | `claude -p "query"`                                                                                |
| `--append-system-prompt`         | Append to system prompt (only with `--print`)                   | `claude --append-system-prompt "Custom instruction"`                                               |
| `--output-format`                | Specify output format (`text`, `json`, `stream-json`)           | `claude -p "query" --output-format json`                                                           |
| `--input-format`                 | Specify input format (`text`, `stream-json`)                    | `claude -p --input-format stream-json`                                                             |
| `--include-partial-messages`     | Include partial streaming events in output                      | `claude -p --output-format stream-json --include-partial-messages "query"`                         |
| `--verbose`                      | Enable verbose logging                                          | `claude --verbose`                                                                                 |
| `--max-turns`                    | Limit the number of agentic turns                               | `claude -p --max-turns 3 "query"`                                                                  |
| `--model`                        | Sets the model for the current session                          | `claude --model claude-sonnet-4-5-20250929`                                                        |
| `--permission-mode`              | Begin in a specified permission mode                            | `claude --permission-mode plan`                                                                    |
| `--permission-prompt-tool`       | MCP tool to handle permission prompts in non-interactive mode   | `claude -p --permission-prompt-tool mcp_auth_tool "query"`                                         |
| `--resume`                       | Resume a specific session by ID                                 | `claude --resume abc123 "query"`                                                                   |
| `--continue`                     | Load the most recent conversation                               | `claude --continue`                                                                                |
| `--dangerously-skip-permissions` | Skip permission prompts (use with caution)                      | `claude --dangerously-skip-permissions`                                                            |

## Agents flag format

The `--agents` flag accepts a JSON object that defines custom subagents:

| Field         | Required | Description                                                                 |
| :------------ | :------- | :-------------------------------------------------------------------------- |
| `description` | Yes      | Natural language description of when the subagent should be invoked         |
| `prompt`      | Yes      | The system prompt that guides the subagent's behavior                       |
| `tools`       | No       | Array of specific tools (e.g., `["Read", "Edit", "Bash"]`)                  |
| `model`       | No       | Model alias to use: `sonnet`, `opus`, or `haiku`                            |

Example:

```bash
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

For more details on creating and using subagents, see [subagents.md](subagents.md).

## Related documentation

* [headless.md](headless.md) - Headless mode and programmatic usage
* [commands.md](commands.md) - Interactive session commands
* [configuration.md](configuration.md) - Configuration options
* [permissions.md](permissions.md) - Permission modes
