# Slash commands

> Control Claude's behavior during an interactive session with slash commands.

## Built-in slash commands

| Command                   | Purpose                                                          |
| :------------------------ | :--------------------------------------------------------------- |
| `/add-dir`                | Add additional working directories                               |
| `/agents`                 | Manage custom AI subagents for specialized tasks                 |
| `/bug`                    | Report bugs                                                      |
| `/clear`                  | Clear conversation history                                       |
| `/compact [instructions]` | Compact conversation with optional focus instructions            |
| `/config`                 | Open the Settings interface                                      |
| `/cost`                   | Show token usage statistics                                      |
| `/doctor`                 | Check health of Claude Code installation                         |
| `/help`                   | Get usage help                                                   |
| `/init`                   | Initialize project with CLAUDE.md guide                          |
| `/login`                  | Switch Anthropic accounts                                        |
| `/logout`                 | Sign out from your Anthropic account                             |
| `/mcp`                    | Manage MCP server connections and OAuth authentication           |
| `/memory`                 | Edit CLAUDE.md memory files                                      |
| `/model`                  | Select or change the AI model                                    |
| `/permissions`            | View or update permissions                                       |
| `/pr_comments`            | View pull request comments                                       |
| `/review`                 | Request code review                                              |
| `/sandbox`                | Enable sandboxed bash tool                                       |
| `/rewind`                 | Rewind the conversation and/or code                              |
| `/status`                 | Show version, model, account, and connectivity                   |
| `/terminal-setup`         | Install Shift+Enter key binding for newlines                     |
| `/usage`                  | Show plan usage limits and rate limit status                     |
| `/vim`                    | Enter vim mode                                                   |

## Custom slash commands

Custom slash commands allow you to define frequently-used prompts as Markdown files that Claude Code can execute.

### Syntax

```
/<command-name> [arguments]
```

### Command types

#### Project commands

Commands stored in your repository and shared with your team. Show "(project)" in `/help`.

**Location**: `.claude/commands/`

Example - creating the `/optimize` command:

```bash
mkdir -p .claude/commands
echo "Analyze this code for performance issues and suggest optimizations:" > .claude/commands/optimize.md
```

#### Personal commands

Commands available across all your projects. Show "(user)" in `/help`.

**Location**: `~/.claude/commands/`

Example - creating the `/security-review` command:

```bash
mkdir -p ~/.claude/commands
echo "Review this code for security vulnerabilities:" > ~/.claude/commands/security-review.md
```

### Features

#### Namespacing

Organize commands in subdirectories. The subdirectories are used for organization and appear in the command description, but do not affect the command name.

For example, `.claude/commands/frontend/component.md` creates `/component` with description showing "(project:frontend)".

#### Arguments

Pass dynamic values to commands using argument placeholders:

##### All arguments with `$ARGUMENTS`

```bash
# Command definition
echo 'Fix issue #$ARGUMENTS following our coding standards' > .claude/commands/fix-issue.md

# Usage
> /fix-issue 123 high-priority
# $ARGUMENTS becomes: "123 high-priority"
```

##### Individual arguments with `$1`, `$2`, etc.

```bash
# Command definition
echo 'Review PR #$1 with priority $2 and assign to $3' > .claude/commands/review-pr.md

# Usage
> /review-pr 456 high alice
# $1 becomes "456", $2 becomes "high", $3 becomes "alice"
```

#### Bash command execution

Execute bash commands before the slash command runs using the `!` prefix. You must include `allowed-tools` with the `Bash` tool:

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
description: Create a git commit
---

## Context

- Current git status: !`git status`
- Current git diff: !`git diff HEAD`
- Current branch: !`git branch --show-current`
- Recent commits: !`git log --oneline -10`

## Your task

Based on the above changes, create a single git commit.
```

#### File references

Include file contents in commands using the `@` prefix:

```markdown
# Reference a specific file
Review the implementation in @src/utils/helpers.js

# Reference multiple files
Compare @src/old-version.js with @src/new-version.js
```

#### Thinking mode

Slash commands can trigger extended thinking by including extended thinking keywords.

### Frontmatter

Command files support frontmatter for metadata:

| Frontmatter                | Purpose                                                  | Default                             |
| :------------------------- | :------------------------------------------------------- | :---------------------------------- |
| `allowed-tools`            | List of tools the command can use                        | Inherits from the conversation      |
| `argument-hint`            | Arguments expected for the slash command                 | None                                |
| `description`              | Brief description of the command                         | Uses the first line from the prompt |
| `model`                    | Specific model string                                    | Inherits from the conversation      |
| `disable-model-invocation` | Whether to prevent SlashCommand tool from calling this   | false                               |

Example:

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git commit:*)
argument-hint: [message]
description: Create a git commit
model: claude-3-5-haiku-20241022
---

Create a git commit with message: $ARGUMENTS
```

## Plugin commands

Plugins can provide custom slash commands. See [plugins.md](plugins.md) for details.

## Related documentation

* [cli-reference.md](cli-reference.md) - CLI commands and flags
* [plugins.md](plugins.md) - Create plugin commands
* [configuration.md](configuration.md) - Settings and configuration
