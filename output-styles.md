# Output styles

> [DEPRECATED] Adapt Claude Code for uses beyond software engineering

**Warning**: Output styles are **DEPRECATED**. On **November 5, 2025** or later, user-level output style files will be automatically converted to plugins and the output styles feature will stop being supported. Use plugins instead.

## Deprecation timeline

As of **November 5, 2025**, Claude Code will:

* Automatically convert user-level output style files (`~/.claude/output-styles`) to plugins
* Stop supporting the output styles feature
* Remove the `/output-style` command and related functionality

**What you need to do:**

* Migrate to plugins before November 5, 2025 for a smoother transition
* Review the migration guide below to understand your options

## Alternative: Use plugins instead

Plugins provide more powerful and flexible ways to customize Claude Code's behavior.

### Installing a plugin

To install a plugin like `explanatory-output-style`:

```shell
/plugin marketplace add anthropics/claude-code
/plugin install explanatory-output-style@claude-code-plugins
/exit
```

To disable:

```shell
/plugin manage explanatory-output-style@claude-code-plugins
# Follow the prompts to disable
/exit
```

For more details on plugins, see [plugins.md](plugins.md).

## Migration guide

Output styles directly modified Claude Code's system prompt. Here's how to achieve similar effects with hooks and subagents:

### Use SessionStart hooks for context injection

If you used output styles to add context at the start of sessions, use SessionStart hooks instead.

The hook's output (stdout) is added to the conversation context. You can also:

* Run scripts that dynamically generate context
* Load project-specific information

**Note**: SessionStart hooks, just like CLAUDE.md, do not change the system prompt.

See [hooks.md](hooks.md) for details.

### Use Subagents for different system prompts

If you used output styles to change Claude's behavior for specific tasks, use Subagents instead.

Subagents are specialized AI assistants with:

* Custom system prompts (must be in a separate context window from main loop)
* Specific tool access permissions
* Optional model to use, if not the main loop model

See [subagents.md](subagents.md) for details.

## Related documentation

* [plugins.md](plugins.md) - Create and use plugins
* [hooks.md](hooks.md) - Use hooks for context injection
* [subagents.md](subagents.md) - Use subagents for custom system prompts
