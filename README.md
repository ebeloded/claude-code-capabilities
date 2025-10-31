# Claude Code Capabilities Skill

A comprehensive reference skill for all Claude Code extensibility features, including subagents, skills, plugins, commands, hooks, MCP servers, permissions, and automation workflows.

## What is this?

This is a **Claude Code skill** that provides detailed documentation about Claude Code's capabilities. When installed, Claude can automatically reference this documentation to help you understand and implement various extensibility features.

## Features Documented

This skill covers:

- **Subagents** - Specialized AI assistants for focused tasks
- **Skills** - Model-invoked modular capabilities
- **Plugins** - Packaged extensions for distribution
- **Commands** - Custom slash commands for frequent tasks
- **Hooks** - Event-driven automation
- **MCP Servers** - External tool integration
- **Permissions** - Security and access control
- **Configuration** - Fine-grained settings
- **Headless Mode** - Programmatic Claude Code execution
- **GitHub Actions** - CI/CD integration

## Installation

### Option 1: Install as a Skill (Recommended)

```bash
# Clone this repository to your Claude skills directory
git clone https://github.com/yourusername/claude-code-capabilities.git ~/.claude/skills/claude-code-capabilities
```

Once installed, Claude will automatically use this skill when you ask questions about Claude Code capabilities.

### Option 2: Use as Reference Documentation

Simply browse the markdown files in this repository:

- [`SKILL.md`](SKILL.md) - Start here for an overview
- [`subagents.md`](subagents.md) - Creating custom subagents
- [`skills.md`](skills.md) - Creating skills
- [`plugins.md`](plugins.md) - Plugin development
- [`commands.md`](commands.md) - Custom slash commands
- [`hooks.md`](hooks.md) - Event-driven automation
- [`mcp.md`](mcp.md) - MCP server integration
- [`permissions.md`](permissions.md) - Security and permissions
- [`configuration.md`](configuration.md) - Settings and configuration
- [`headless.md`](headless.md) - Programmatic usage
- [`github-actions.md`](github-actions.md) - CI/CD integration
- [`cli-reference.md`](cli-reference.md) - CLI commands and flags
- [`examples/`](examples/) - Practical examples

## Usage

Once installed as a skill, simply ask Claude Code questions like:

- "How do I create a custom subagent?"
- "What's the difference between skills and commands?"
- "How can I add a pre-commit hook?"
- "Show me examples of MCP server integration"

Claude will automatically reference this documentation to provide accurate, detailed answers.

## Documentation Structure

```
.
├── SKILL.md                      # Main overview and quick reference
├── subagents.md                  # Specialized AI assistants
├── skills.md                     # Model-invoked capabilities
├── plugins.md                    # Packaged extensions
├── commands.md                   # Custom slash commands
├── hooks.md                      # Event-driven automation
├── mcp.md                        # External tool integration
├── permissions.md                # Security and access control
├── configuration.md              # Settings and configuration
├── headless.md                   # Programmatic usage
├── github-actions.md             # CI/CD integration
├── cli-reference.md              # CLI commands reference
├── output-styles.md              # Output styles (deprecated)
└── examples/
    ├── subagent-examples.md      # Subagent patterns
    ├── hook-examples.md          # Hook configurations
    └── plugin-examples.md        # Plugin development
```

## Quick Start Guide

### For New Users

1. Start with [`SKILL.md`](SKILL.md) for an overview
2. Read [`configuration.md`](configuration.md) to understand settings
3. Try creating a simple command with [`commands.md`](commands.md)

### For Automation

1. Learn about [`hooks.md`](hooks.md) for event-driven automation
2. Check [`examples/hook-examples.md`](examples/hook-examples.md) for practical patterns
3. Explore [`github-actions.md`](github-actions.md) for CI/CD

### For Teams

1. Read [`plugins.md`](plugins.md) to package your customizations
2. Review [`permissions.md`](permissions.md) for security
3. Use [`headless.md`](headless.md) for scripting

## Contributing

Contributions are welcome! If you find errors or want to add more examples:

1. Fork this repository
2. Make your changes
3. Submit a pull request

## License

MIT License - Feel free to use and modify as needed.

## About Claude Code

Claude Code is Anthropic's official CLI for Claude, an AI assistant that helps with software engineering tasks. Learn more at [docs.claude.com](https://docs.claude.com).

## Support

- [Claude Code Documentation](https://docs.claude.com/claude-code)
- [GitHub Issues](https://github.com/anthropics/claude-code/issues)
- [Community Discussions](https://github.com/anthropics/claude-code/discussions)
