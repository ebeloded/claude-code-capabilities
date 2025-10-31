# Plugin Examples

Practical examples for creating Claude Code plugins.

## Simple command plugin

A basic plugin that adds a greeting command:

**Directory structure:**
```
greeting-plugin/
├── .claude-plugin/
│   └── plugin.json
└── commands/
    └── greet.md
```

**.claude-plugin/plugin.json:**
```json
{
  "name": "greeting-plugin",
  "description": "Simple greeting command",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  }
}
```

**commands/greet.md:**
```markdown
---
description: Greet the user warmly
---

Greet the user with enthusiasm and ask how you can help them today!
```

## Plugin with subagent

A plugin that adds a code review subagent:

**Directory structure:**
```
code-review-plugin/
├── .claude-plugin/
│   └── plugin.json
└── agents/
    └── reviewer.md
```

**agents/reviewer.md:**
```markdown
---
name: code-reviewer
description: Expert code reviewer for quality and security. Use after code changes.
tools: Read, Grep, Glob, Bash
---

You are a senior code reviewer. Review code for:
- Code quality and readability
- Security vulnerabilities
- Performance issues
- Best practices
- Test coverage

Provide actionable feedback with examples.
```

## Plugin with Skills

A plugin that adds PDF processing skills:

**Directory structure:**
```
pdf-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── pdf-processing/
        ├── SKILL.md
        └── scripts/
            └── extract-text.py
```

**skills/pdf-processing/SKILL.md:**
```markdown
---
name: pdf-processing
description: Extract text, fill forms, merge PDFs. Use when working with PDF files.
---

# PDF Processing

Extract text from PDFs, fill forms, and merge documents.

## Requirements

Install dependencies:
```bash
pip install pypdf pdfplumber
```

## Usage

For text extraction, use the helper script:
```bash
python scripts/extract-text.py document.pdf
```
```

## Plugin with hooks

A plugin that adds automatic formatting:

**Directory structure:**
```
formatter-plugin/
├── .claude-plugin/
│   └── plugin.json
└── hooks/
    ├── hooks.json
    └── scripts/
        └── format.sh
```

**hooks/hooks.json:**
```json
{
  "description": "Automatic code formatting",
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format.sh",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**hooks/scripts/format.sh:**
```bash
#!/bin/bash

file_path=$(jq -r '.tool_input.file_path' 2>/dev/null)

if [ -z "$file_path" ]; then
    exit 0
fi

# Format based on file extension
case "$file_path" in
    *.ts|*.js)
        npx prettier --write "$file_path" 2>/dev/null
        ;;
    *.py)
        black "$file_path" 2>/dev/null
        ;;
    *.go)
        gofmt -w "$file_path" 2>/dev/null
        ;;
esac

exit 0
```

## Comprehensive plugin

A full-featured plugin with multiple components:

**Directory structure:**
```
dev-toolkit-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── test.md
│   └── lint.md
├── agents/
│   ├── debugger.md
│   └── optimizer.md
├── skills/
│   └── testing/
│       └── SKILL.md
└── hooks/
    └── hooks.json
```

**.claude-plugin/plugin.json:**
```json
{
  "name": "dev-toolkit",
  "description": "Comprehensive development toolkit with testing, linting, debugging, and optimization",
  "version": "2.0.0",
  "author": {
    "name": "Dev Team",
    "email": "team@example.com"
  },
  "keywords": ["testing", "linting", "debugging", "optimization"]
}
```

**commands/test.md:**
```markdown
---
allowed-tools: Bash(npm test:*), Bash(pytest:*), Bash(go test:*)
description: Run project tests with coverage
---

Run the appropriate test suite for this project and report coverage results.
```

**commands/lint.md:**
```markdown
---
allowed-tools: Bash(npm run lint:*), Bash(eslint:*), Bash(pylint:*)
description: Run linting checks
---

Run linting checks for the project and fix any auto-fixable issues.
```

## Plugin with MCP integration

A plugin that includes MCP server configuration:

**Directory structure:**
```
github-integration-plugin/
├── .claude-plugin/
│   └── plugin.json
├── .mcp.json
└── commands/
    └── create-issue.md
```

**.mcp.json:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

**commands/create-issue.md:**
```markdown
---
allowed-tools: mcp__github__create_issue
description: Create a GitHub issue [title] [body]
argument-hint: [title] [body]
---

Create a GitHub issue with the title "$1" and body "$2" in the current repository.
```

## Testing plugins locally

Create a local marketplace for testing:

**marketplace structure:**
```
test-marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── my-plugin/
    └── (plugin files)
```

**.claude-plugin/marketplace.json:**
```json
{
  "name": "test-marketplace",
  "owner": {
    "name": "Developer"
  },
  "plugins": [
    {
      "name": "my-plugin",
      "source": "./my-plugin",
      "description": "Plugin under development"
    }
  ]
}
```

**Testing workflow:**
```bash
# Add marketplace
claude
> /plugin marketplace add ./test-marketplace

# Install plugin
> /plugin install my-plugin@test-marketplace

# Test changes
> /plugin uninstall my-plugin@test-marketplace
> /plugin install my-plugin@test-marketplace
```

## Related documentation

For more information on creating plugins, see [plugins.md](../plugins.md).
