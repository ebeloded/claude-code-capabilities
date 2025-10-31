# Claude Code GitHub Actions

> Integrate Claude Code into your development workflow with GitHub Actions

Claude Code GitHub Actions brings AI-powered automation to your GitHub workflow. With a simple `@claude` mention in any PR or issue, Claude can analyze your code, create pull requests, implement features, and fix bugs.

## Quick setup

The easiest way to set up is through Claude Code in the terminal:

```
/install-github-app
```

This command guides you through setting up the GitHub app and required secrets.

**Note**: You must be a repository admin to install the GitHub app and add secrets.

## Manual setup

1. **Install the Claude GitHub app** to your repository: https://github.com/apps/claude

   Required permissions:
   * **Contents**: Read & write
   * **Issues**: Read & write
   * **Pull requests**: Read & write

2. **Add ANTHROPIC_API_KEY** to your repository secrets

3. **Copy the workflow file** from the examples into `.github/workflows/`

## Example use cases

### Basic workflow

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Responds to @claude mentions in comments
```

### Using slash commands

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "/review"
          claude_args: "--max-turns 5"
```

### Custom automation with prompts

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"
jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Generate a summary of yesterday's commits and open issues"
          claude_args: "--model claude-opus-4-1-20250805"
```

### Common use cases

In issue or PR comments:

```
@claude implement this feature based on the issue description
@claude how should I implement user authentication for this endpoint?
@claude fix the TypeError in the user dashboard component
```

## Configuration

The action accepts the following inputs:

| Input                 | Description                                          | Required |
| :-------------------- | :--------------------------------------------------- | :------- |
| `anthropic_api_key`   | Anthropic API key for authentication                 | Yes      |
| `prompt`              | Direct prompt to execute                             | No       |
| `claude_args`         | Additional CLI arguments (e.g., `--max-turns 5`)     | No       |
| `settings`            | JSON settings configuration                          | No       |

## Upgrading from Beta

Claude Code GitHub Actions v1.0 introduces breaking changes. To upgrade from beta:

1. **Update the action version**: Change `@beta` to `@v1`
2. **Remove mode configuration**: Delete `mode: "tag"` or `mode: "agent"`
3. **Update prompt inputs**: Replace `direct_prompt` with `prompt`
4. **Move CLI options**: Convert to `claude_args`

**Before (Beta)**:

```yaml
- uses: anthropics/claude-code-action@beta
  with:
    mode: "tag"
    direct_prompt: "Review this PR for security issues"
    custom_instructions: "Follow our coding standards"
    max_turns: "10"
```

**After (v1.0)**:

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    prompt: "Review this PR for security issues"
    claude_args: |
      --system-prompt "Follow our coding standards"
      --max-turns 10
```

## Related documentation

* [headless.md](headless.md) - Headless mode and CLI options
* [cli-reference.md](cli-reference.md) - Complete CLI reference
* [commands.md](commands.md) - Slash commands
