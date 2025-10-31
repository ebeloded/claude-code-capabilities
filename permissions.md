# Identity and Access Management

> Learn how to configure user authentication, authorization, and access controls for Claude Code.

## Access control and permissions

Claude Code uses a tiered permission system to balance power and safety.

### Permission system

| Tool Type         | Example              | Approval Required | "Yes, don't ask again" Behavior               |
| :---------------- | :------------------- | :---------------- | :-------------------------------------------- |
| Read-only         | File reads, LS, Grep | No                | N/A                                           |
| Bash Commands     | Shell execution      | Yes               | Permanently per project directory and command |
| File Modification | Edit/write files     | Yes               | Until session end                             |

### Configuring permissions

View & manage Claude Code's tool permissions with `/permissions`. This UI lists all permission rules and the settings.json file they are sourced from.

* **Allow** rules allow Claude Code to use the specified tool without further manual approval.
* **Ask** rules ask the user for confirmation whenever Claude Code tries to use the specified tool. Ask rules take precedence over allow rules.
* **Deny** rules prevent Claude Code from using the specified tool. Deny rules take precedence over allow and ask rules.
* **Additional directories** extend Claude's file access to directories beyond the initial working directory.
* **Default mode** controls Claude's permission behavior when encountering new requests.

Permission rules use the format: `Tool` or `Tool(optional-specifier)`

A rule that is just the tool name matches any use of that tool. For example, adding `Bash` to the list of allow rules would allow Claude Code to use the Bash tool without requiring user approval.

## Permission modes

Claude Code supports several permission modes that can be set as the `defaultMode` in settings files:

| Mode                | Description                                                                  |
| :------------------ | :--------------------------------------------------------------------------- |
| `default`           | Standard behavior - prompts for permission on first use of each tool         |
| `acceptEdits`       | Automatically accepts file edit permissions for the session                  |
| `plan`              | Plan Mode - Claude can analyze but not modify files or execute commands      |
| `bypassPermissions` | Skips all permission prompts (requires safe environment - use with caution)  |

## Working directories

By default, Claude has access to files in the directory where it was launched. You can extend this access:

* **During startup**: Use `--add-dir <path>` CLI argument
* **During session**: Use `/add-dir` slash command
* **Persistent configuration**: Add to `additionalDirectories` in settings files

Files in additional directories follow the same permission rules as the original working directory.

## Tool-specific permission rules

### Bash

* `Bash(npm run build)` - Matches the exact Bash command `npm run build`
* `Bash(npm run test:*)` - Matches Bash commands starting with `npm run test`
* `Bash(curl http://site.com/:*)` - Matches curl commands that start with exactly `curl http://site.com/`

**Note**: Claude Code is aware of shell operators (like `&&`) so a prefix match rule like `Bash(safe-cmd:*)` won't give it permission to run the command `safe-cmd && other-cmd`

**Important limitations of Bash permission patterns**:

1. Uses **prefix matches**, not regex or glob patterns
2. The wildcard `:*` only works at the end of a pattern
3. Patterns can be bypassed in many ways (options before URL, different protocol, redirects, variables, extra spaces)

For more reliable URL filtering, consider:
* Using the WebFetch tool with `WebFetch(domain:github.com)` permission
* Instructing Claude Code about your allowed curl patterns via CLAUDE.md
* Using hooks for custom permission validation

### Read & Edit

`Edit` rules apply to all built-in tools that edit files. Claude will make a best-effort attempt to apply `Read` rules to all built-in tools that read files like Grep, Glob, and LS.

Read & Edit rules follow the gitignore specification with four distinct pattern types:

| Pattern            | Meaning                                | Example                          | Matches                            |
| ------------------ | -------------------------------------- | -------------------------------- | ---------------------------------- |
| `//path`           | **Absolute** path from filesystem root | `Read(//Users/alice/secrets/**)` | `/Users/alice/secrets/**`          |
| `~/path`           | Path from **home** directory           | `Read(~/Documents/*.pdf)`        | `/Users/alice/Documents/*.pdf`     |
| `/path`            | Path **relative to settings file**     | `Edit(/src/**/*.ts)`             | `<settings file path>/src/**/*.ts` |
| `path` or `./path` | Path **relative to current directory** | `Read(*.env)`                    | `<cwd>/*.env`                      |

**Warning**: A pattern like `/Users/alice/file` is NOT an absolute path - it's relative to your settings file! Use `//Users/alice/file` for absolute paths.

Examples:
* `Edit(/docs/**)` - Edits in `<project>/docs/` (NOT `/docs/`!)
* `Read(~/.zshrc)` - Reads your home directory's `.zshrc`
* `Edit(//tmp/scratch.txt)` - Edits the absolute path `/tmp/scratch.txt`
* `Read(src/**)` - Reads from `<current-directory>/src/`

### WebFetch

* `WebFetch(domain:example.com)` - Matches fetch requests to example.com

### MCP

* `mcp__puppeteer` - Matches any tool provided by the `puppeteer` server
* `mcp__puppeteer__puppeteer_navigate` - Matches the `puppeteer_navigate` tool provided by the `puppeteer` server

**Warning**: Unlike other permission types, MCP permissions do NOT support wildcards (`*`).

To approve all tools from an MCP server:
* ✅ Use: `mcp__github` (approves ALL GitHub tools)
* ❌ Don't use: `mcp__github__*` (wildcards are not supported)

To approve specific tools only, list each one:
* ✅ Use: `mcp__github__get_issue`
* ✅ Use: `mcp__github__list_issues`

## Additional permission control with hooks

Claude Code hooks provide a way to register custom shell commands to perform permission evaluation at runtime. When Claude Code makes a tool call, PreToolUse hooks run before the permission system runs, and the hook output can determine whether to approve or deny the tool call in place of the permission system.

See [hooks.md](hooks.md) for details.

## Settings files and precedence

Permission settings are configured in settings files:

* `~/.claude/settings.json` - User settings
* `.claude/settings.json` - Project settings (shared with team)
* `.claude/settings.local.json` - Local project settings (not committed)
* Enterprise managed policy settings

### Settings precedence

When multiple settings sources exist, they are applied in the following order (highest to lowest precedence):

1. Enterprise policies
2. Command line arguments
3. Local project settings (`.claude/settings.local.json`)
4. Shared project settings (`.claude/settings.json`)
5. User settings (`~/.claude/settings.json`)

## Example configuration

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test:*)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl:*)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ],
    "additionalDirectories": ["../docs/"],
    "defaultMode": "acceptEdits"
  }
}
```

## Excluding sensitive files

To prevent Claude Code from accessing files containing sensitive information:

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)"
    ]
  }
}
```

Files matching these patterns will be completely invisible to Claude Code.

## Related documentation

* [configuration.md](configuration.md) - Settings files and configuration options
* [hooks.md](hooks.md) - Custom permission validation with hooks
* [mcp.md](mcp.md) - MCP tool permissions
