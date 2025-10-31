# Claude Code settings

> Configure Claude Code with global and project-level settings, and environment variables.

Claude Code offers a variety of settings to configure its behavior. You can configure Claude Code by running the `/config` command in the interactive REPL.

## Settings files

The `settings.json` file is the official mechanism for configuring Claude Code through hierarchical settings:

* **User settings**: `~/.claude/settings.json` - Apply to all projects
* **Project settings**:
  * `.claude/settings.json` - Checked into source control and shared with your team
  * `.claude/settings.local.json` - Not checked in, for personal preferences
* **Enterprise managed policy settings**: Take precedence over user and project settings
  * macOS: `/Library/Application Support/ClaudeCode/managed-settings.json`
  * Linux/WSL: `/etc/claude-code/managed-settings.json`
  * Windows: `C:\ProgramData\ClaudeCode\managed-settings.json`

### Example settings.json

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
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  }
}
```

## Available settings

| Key                          | Description                                                                  | Example                                                     |
| :--------------------------- | :--------------------------------------------------------------------------- | :---------------------------------------------------------- |
| `apiKeyHelper`               | Custom script to generate an auth value                                      | `/bin/generate_temp_api_key.sh`                             |
| `cleanupPeriodDays`          | How long to retain chat transcripts (default: 30 days)                       | `20`                                                        |
| `env`                        | Environment variables applied to every session                               | `{"FOO": "bar"}`                                            |
| `includeCoAuthoredBy`        | Include `co-authored-by Claude` byline in git commits (default: `true`)      | `false`                                                     |
| `permissions`                | Permission rules for tools and file access                                   | See [permissions.md](permissions.md)                        |
| `hooks`                      | Custom commands to run before or after tool executions                       | See [hooks.md](hooks.md)                                    |
| `disableAllHooks`            | Disable all hooks                                                            | `true`                                                      |
| `model`                      | Override the default model                                                   | `"claude-sonnet-4-5-20250929"`                              |
| `statusLine`                 | Configure a custom status line                                               | `{"type": "command", "command": "~/.claude/statusline.sh"}` |
| `outputStyle`                | Configure an output style (deprecated)                                       | `"Explanatory"`                                             |
| `forceLoginMethod`           | Restrict login method (`claudeai` or `console`)                              | `claudeai`                                                  |
| `forceLoginOrgUUID`          | Auto-select organization during login                                        | `"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`                    |
| `enableAllProjectMcpServers` | Automatically approve all MCP servers in `.mcp.json`                         | `true`                                                      |
| `enabledMcpjsonServers`      | List of specific MCP servers to approve                                      | `["memory", "github"]`                                      |
| `disabledMcpjsonServers`     | List of specific MCP servers to reject                                       | `["filesystem"]`                                            |

## Permission settings

| Keys                           | Description                                          | Example                                                                |
| :----------------------------- | :--------------------------------------------------- | :--------------------------------------------------------------------- |
| `allow`                        | Array of permission rules to allow tool use          | `[ "Bash(git diff:*)" ]`                                               |
| `ask`                          | Array of permission rules to ask for confirmation    | `[ "Bash(git push:*)" ]`                                               |
| `deny`                         | Array of permission rules to deny tool use           | `[ "WebFetch", "Bash(curl:*)", "Read(./.env)", "Read(./secrets/**)" ]` |
| `additionalDirectories`        | Additional working directories Claude has access to  | `[ "../docs/" ]`                                                       |
| `defaultMode`                  | Default permission mode when opening Claude Code     | `"acceptEdits"`                                                        |
| `disableBypassPermissionsMode` | Prevent `bypassPermissions` mode from being activated| `"disable"`                                                            |

For detailed permission configuration, see [permissions.md](permissions.md).

## Sandbox settings

Configure advanced sandboxing behavior. Sandboxing isolates bash commands from your filesystem and network.

**Note**: Filesystem and network restrictions are configured via Read, Edit, and WebFetch permission rules, not via these sandbox settings.

| Keys                        | Description                                                       | Example                   |
| :-------------------------- | :---------------------------------------------------------------- | :------------------------ |
| `enabled`                   | Enable bash sandboxing (macOS/Linux only). Default: false         | `true`                    |
| `autoAllowBashIfSandboxed`  | Auto-approve bash commands when sandboxed. Default: true          | `true`                    |
| `excludedCommands`          | Commands that should run outside of the sandbox                   | `["git", "docker"]`       |
| `allowUnsandboxedCommands`  | Allow commands to run outside via escape hatch. Default: true     | `false`                   |
| `network.allowUnixSockets`  | Unix socket paths accessible in sandbox                           | `["~/.ssh/agent-socket"]` |
| `network.allowLocalBinding` | Allow binding to localhost ports (MacOS only). Default: false     | `true`                    |
| `network.httpProxyPort`     | HTTP proxy port for custom proxy                                  | `8080`                    |
| `network.socksProxyPort`    | SOCKS5 proxy port for custom proxy                                | `8081`                    |
| `enableWeakerNestedSandbox` | Enable weaker sandbox for Docker (Linux only). Default: false     | `true`                    |

**Configuration example:**

```json
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker"],
    "network": {
      "allowUnixSockets": [
        "/var/run/docker.sock"
      ],
      "allowLocalBinding": true
    }
  },
  "permissions": {
    "deny": [
      "Read(.envrc)",
      "Read(~/.aws/**)"
    ]
  }
}
```

## Settings precedence

Settings are applied in order of precedence (highest to lowest):

1. **Enterprise managed policies** (`managed-settings.json`)
2. **Command line arguments**
3. **Local project settings** (`.claude/settings.local.json`)
4. **Shared project settings** (`.claude/settings.json`)
5. **User settings** (`~/.claude/settings.json`)

This hierarchy ensures that enterprise security policies are always enforced while allowing teams and individuals to customize their experience.

## Excluding sensitive files

To prevent Claude Code from accessing files containing sensitive information:

```json
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./build)"
    ]
  }
}
```

This replaces the deprecated `ignorePatterns` configuration. Files matching these patterns will be completely invisible to Claude Code.

## Subagent configuration

Subagents can have their own model configuration. See [subagents.md](subagents.md) for details.

## CLAUDE.md memory files

CLAUDE.md files contain instructions and context that Claude loads at startup. These are different from settings files - they provide prompts and context rather than configuration.

## Related documentation

* [permissions.md](permissions.md) - Detailed permission configuration
* [hooks.md](hooks.md) - Hook configuration
* [mcp.md](mcp.md) - MCP server configuration
* [subagents.md](subagents.md) - Subagent configuration
* [cli-reference.md](cli-reference.md) - Command-line options
