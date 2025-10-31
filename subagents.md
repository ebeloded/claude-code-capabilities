# Subagents

> Create and use specialized AI subagents in Claude Code for task-specific workflows and improved context management.

Custom subagents in Claude Code are specialized AI assistants that can be invoked to handle specific types of tasks. They enable more efficient problem-solving by providing task-specific configurations with customized system prompts, tools and a separate context window.

## What are subagents?

Subagents are pre-configured AI personalities that Claude Code can delegate tasks to. Each subagent:

* Has a specific purpose and expertise area
* Uses its own context window separate from the main conversation
* Can be configured with specific tools it's allowed to use
* Includes a custom system prompt that guides its behavior

When Claude Code encounters a task that matches a subagent's expertise, it can delegate that task to the specialized subagent, which works independently and returns results.

## Key benefits

* **Context preservation**: Each subagent operates in its own context, preventing pollution of the main conversation and keeping it focused on high-level objectives.
* **Specialized expertise**: Subagents can be fine-tuned with detailed instructions for specific domains, leading to higher success rates on designated tasks.
* **Reusability**: Once created, subagents can be used across different projects and shared with your team for consistent workflows.
* **Flexible permissions**: Each subagent can have different tool access levels, allowing you to limit powerful tools to specific subagent types.

## Quick start

To create your first subagent:

1. **Open the subagents interface**: Run `/agents`
2. **Select 'Create New Agent'**: Choose whether to create a project-level or user-level subagent
3. **Define the subagent**:
   * **Recommended**: Generate with Claude first, then customize to make it yours
   * Describe your subagent in detail and when it should be used
   * Select the tools you want to grant access to (or leave blank to inherit all tools)
   * The interface shows all available tools, making selection easy
   * If you're generating with Claude, you can also edit the system prompt in your own editor by pressing `e`
4. **Save and use**: Your subagent is now available! Claude will use it automatically when appropriate, or you can invoke it explicitly:
   ```
   > Use the code-reviewer subagent to check my recent changes
   ```

## Subagent configuration

### File locations

Subagents are stored as Markdown files with YAML frontmatter in two possible locations:

| Type                  | Location            | Scope                         | Priority |
| :-------------------- | :------------------ | :---------------------------- | :------- |
| **Project subagents** | `.claude/agents/`   | Available in current project  | Highest  |
| **User subagents**    | `~/.claude/agents/` | Available across all projects | Lower    |

When subagent names conflict, project-level subagents take precedence over user-level subagents.

### Plugin agents

Plugins can provide custom subagents that integrate seamlessly with Claude Code. Plugin agents work identically to user-defined agents and appear in the `/agents` interface.

**Plugin agent locations**: Plugins include agents in their `agents/` directory (or custom paths specified in the plugin manifest).

**Using plugin agents**:

* Plugin agents appear in `/agents` alongside your custom agents
* Can be invoked explicitly: "Use the code-reviewer agent from the security-plugin"
* Can be invoked automatically by Claude when appropriate
* Can be managed (viewed, inspected) through `/agents` interface

See [plugins.md](plugins.md) for details on creating plugin agents.

### CLI-based configuration

You can also define subagents dynamically using the `--agents` CLI flag, which accepts a JSON object:

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

**Priority**: CLI-defined subagents have lower priority than project-level subagents but higher priority than user-level subagents.

**Use case**: This approach is useful for:

* Quick testing of subagent configurations
* Session-specific subagents that don't need to be saved
* Automation scripts that need custom subagents
* Sharing subagent definitions in documentation or scripts

For detailed information about the JSON format and all available options, see [cli-reference.md](cli-reference.md).

### File format

Each subagent is defined in a Markdown file with this structure:

```markdown
---
name: your-sub-agent-name
description: Description of when this subagent should be invoked
tools: tool1, tool2, tool3  # Optional - inherits all tools if omitted
model: sonnet  # Optional - specify model alias or 'inherit'
---

Your subagent's system prompt goes here. This can be multiple paragraphs
and should clearly define the subagent's role, capabilities, and approach
to solving problems.

Include specific instructions, best practices, and any constraints
the subagent should follow.
```

#### Configuration fields

| Field         | Required | Description                                                                                                                                                                                                                      |
| :------------ | :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`        | Yes      | Unique identifier using lowercase letters and hyphens                                                                                                                                                                            |
| `description` | Yes      | Natural language description of the subagent's purpose                                                                                                                                                                           |
| `tools`       | No       | Comma-separated list of specific tools. If omitted, inherits all tools from the main thread                                                                                                                                      |
| `model`       | No       | Model to use for this subagent. Can be a model alias (`sonnet`, `opus`, `haiku`) or `'inherit'` to use the main conversation's model. If omitted, defaults to the configured subagent model |

### Model selection

The `model` field allows you to control which AI model the subagent uses:

* **Model alias**: Use one of the available aliases: `sonnet`, `opus`, or `haiku`
* **`'inherit'`**: Use the same model as the main conversation (useful for consistency)
* **Omitted**: If not specified, uses the default model configured for subagents (`sonnet`)

Using `'inherit'` is particularly useful when you want your subagents to adapt to the model choice of the main conversation, ensuring consistent capabilities and response style throughout your session.

### Available tools

Subagents can be granted access to any of Claude Code's internal tools. See [configuration.md](configuration.md) for a complete list of available tools.

**Recommended:** Use the `/agents` command to modify tool access - it provides an interactive interface that lists all available tools, including any connected MCP server tools, making it easier to select the ones you need.

You have two options for configuring tools:

* **Omit the `tools` field** to inherit all tools from the main thread (default), including MCP tools
* **Specify individual tools** as a comma-separated list for more granular control (can be edited manually or via `/agents`)

**MCP Tools**: Subagents can access MCP tools from configured MCP servers. When the `tools` field is omitted, subagents inherit all MCP tools available to the main thread.

## Managing subagents

### Using the /agents command (Recommended)

The `/agents` command provides a comprehensive interface for subagent management:

```
/agents
```

This opens an interactive menu where you can:

* View all available subagents (built-in, user, and project)
* Create new subagents with guided setup
* Edit existing custom subagents, including their tool access
* Delete custom subagents
* See which subagents are active when duplicates exist
* **Easily manage tool permissions** with a complete list of available tools

### Direct file management

You can also manage subagents by working directly with their files:

```bash
# Create a project subagent
mkdir -p .claude/agents
echo '---
name: test-runner
description: Use proactively to run tests and fix failures
---

You are a test automation expert. When you see code changes, proactively run the appropriate tests. If tests fail, analyze the failures and fix them while preserving the original test intent.' > .claude/agents/test-runner.md

# Create a user subagent
mkdir -p ~/.claude/agents
# ... create subagent file
```

## Using subagents effectively

### Automatic delegation

Claude Code proactively delegates tasks based on:

* The task description in your request
* The `description` field in subagent configurations
* Current context and available tools

**Tip**: To encourage more proactive subagent use, include phrases like "use PROACTIVELY" or "MUST BE USED" in your `description` field.

### Explicit invocation

Request a specific subagent by mentioning it in your command:

```
> Use the test-runner subagent to fix failing tests
> Have the code-reviewer subagent look at my recent changes
> Ask the debugger subagent to investigate this error
```

## Best practices

* **Start with Claude-generated agents**: We highly recommend generating your initial subagent with Claude and then iterating on it to make it personally yours. This approach gives you the best results - a solid foundation that you can customize to your specific needs.

* **Design focused subagents**: Create subagents with single, clear responsibilities rather than trying to make one subagent do everything. This improves performance and makes subagents more predictable.

* **Write detailed prompts**: Include specific instructions, examples, and constraints in your system prompts. The more guidance you provide, the better the subagent will perform.

* **Limit tool access**: Only grant tools that are necessary for the subagent's purpose. This improves security and helps the subagent focus on relevant actions.

* **Version control**: Check project subagents into version control so your team can benefit from and improve them collaboratively.

## Advanced usage

### Chaining subagents

For complex workflows, you can chain multiple subagents:

```
> First use the code-analyzer subagent to find performance issues, then use the optimizer subagent to fix them
```

### Dynamic subagent selection

Claude Code intelligently selects subagents based on context. Make your `description` fields specific and action-oriented for best results.

## Performance considerations

* **Context efficiency**: Agents help preserve main context, enabling longer overall sessions
* **Latency**: Subagents start off with a clean slate each time they are invoked and may add latency as they gather context that they require to do their job effectively.

## Related documentation

* [plugins.md](plugins.md) - Extend Claude Code with custom agents through plugins
* [commands.md](commands.md) - Learn about custom slash commands
* [configuration.md](configuration.md) - Configure Claude Code behavior
* [hooks.md](hooks.md) - Automate workflows with event handlers
