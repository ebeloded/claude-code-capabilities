# Plugins

> Extend Claude Code with custom commands, agents, hooks, Skills, and MCP servers through the plugin system.

Plugins let you extend Claude Code with custom functionality that can be shared across projects and teams. Install plugins from marketplaces to add pre-built commands, agents, hooks, Skills, and MCP servers, or create your own to automate your workflows.

## Quickstart

Let's create a simple greeting plugin:

### Create your first plugin

1. **Create the marketplace structure**
   ```bash
   mkdir test-marketplace
   cd test-marketplace
   ```

2. **Create the plugin directory**
   ```bash
   mkdir my-first-plugin
   cd my-first-plugin
   ```

3. **Create the plugin manifest**
   ```bash
   mkdir .claude-plugin
   cat > .claude-plugin/plugin.json << 'EOF'
   {
   "name": "my-first-plugin",
   "description": "A simple greeting plugin to learn the basics",
   "version": "1.0.0",
   "author": {
   "name": "Your Name"
   }
   }
   EOF
   ```

4. **Add a custom command**
   ```bash
   mkdir commands
   cat > commands/hello.md << 'EOF'
   ---
   description: Greet the user with a personalized message
   ---

   # Hello Command

   Greet the user warmly and ask how you can help them today. Make the greeting personal and encouraging.
   EOF
   ```

5. **Create the marketplace manifest**
   ```bash
   cd ..
   mkdir .claude-plugin
   cat > .claude-plugin/marketplace.json << 'EOF'
   {
   "name": "test-marketplace",
   "owner": {
   "name": "Test User"
   },
   "plugins": [
   {
     "name": "my-first-plugin",
     "source": "./my-first-plugin",
     "description": "My first test plugin"
   }
   ]
   }
   EOF
   ```

6. **Install and test your plugin**
   ```bash
   cd ..
   claude
   ```

   Then in Claude:
   ```
   /plugin marketplace add ./test-marketplace
   /plugin install my-first-plugin@test-marketplace
   ```

   Select "Install now". Restart Claude Code, then try:
   ```
   /hello
   ```

## Plugin structure overview

Your plugin follows this basic structure:

```
my-first-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin metadata
├── commands/                 # Custom slash commands (optional)
│   └── hello.md
├── agents/                   # Custom agents (optional)
│   └── helper.md
├── skills/                   # Agent Skills (optional)
│   └── my-skill/
│       └── SKILL.md
└── hooks/                    # Event handlers (optional)
    └── hooks.json
```

**Additional components you can add:**

* **Commands**: Create markdown files in `commands/` directory
* **Agents**: Create agent definitions in `agents/` directory
* **Skills**: Create `SKILL.md` files in `skills/` directory
* **Hooks**: Create `hooks/hooks.json` for event handling
* **MCP servers**: Create `.mcp.json` for external tool integration

## Install and manage plugins

### Add marketplaces

Marketplaces are catalogs of available plugins. Add them to discover and install plugins:

```shell
/plugin marketplace add your-org/claude-plugins
```

```shell
/plugin
```

### Install plugins

#### Via interactive menu (recommended)

```shell
/plugin
```

Select "Browse Plugins" to see available options with descriptions, features, and installation options.

#### Via direct commands

```shell
/plugin install formatter@your-org
```

```shell
/plugin enable plugin-name@marketplace-name
```

```shell
/plugin disable plugin-name@marketplace-name
```

```shell
/plugin uninstall plugin-name@marketplace-name
```

### Verify installation

After installing a plugin:

1. **Check available commands**: Run `/help` to see new commands
2. **Test plugin features**: Try the plugin's commands and features
3. **Review plugin details**: Use `/plugin` → "Manage Plugins" to see what the plugin provides

## Set up team plugin workflows

Configure plugins at the repository level to ensure consistent tooling across your team. When team members trust your repository folder, Claude Code automatically installs specified marketplaces and plugins.

**To set up team plugins:**

1. Add marketplace and plugin configuration to your repository's `.claude/settings.json`
2. Team members trust the repository folder
3. Plugins install automatically for all team members

## Develop more complex plugins

### Add Skills to your plugin

Plugins can include Agent Skills to extend Claude's capabilities. Skills are model-invoked—Claude autonomously uses them based on the task context.

To add Skills to your plugin, create a `skills/` directory at your plugin root and add Skill folders with `SKILL.md` files. Plugin Skills are automatically available when the plugin is installed.

For complete Skill authoring guidance, see [skills.md](skills.md).

### Test your plugins locally

When developing plugins, use a local marketplace to test changes iteratively:

1. **Set up your development structure**
   ```bash
   mkdir dev-marketplace
   cd dev-marketplace
   mkdir my-plugin
   ```

2. **Create the marketplace manifest**
   ```bash
   mkdir .claude-plugin
   cat > .claude-plugin/marketplace.json << 'EOF'
   {
   "name": "dev-marketplace",
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
   EOF
   ```

3. **Install and test**
   ```bash
   cd ..
   claude
   ```

   ```shell
   /plugin marketplace add ./dev-marketplace
   /plugin install my-plugin@dev-marketplace
   ```

   Test your plugin components:
   * Try your commands with `/command-name`
   * Check that agents appear in `/agents`
   * Verify hooks work as expected

4. **Iterate on your plugin**
   ```shell
   /plugin uninstall my-plugin@dev-marketplace
   /plugin install my-plugin@dev-marketplace
   ```

   Repeat this cycle as you develop and refine your plugin.

### Debug plugin issues

If your plugin isn't working as expected:

1. **Check the structure**: Ensure your directories are at the plugin root, not inside `.claude-plugin/`
2. **Test components individually**: Check each command, agent, and hook separately
3. **Use validation and debugging tools**: Check logs and error messages

### Share your plugins

When your plugin is ready to share:

1. **Add documentation**: Include a README.md with installation and usage instructions
2. **Version your plugin**: Use semantic versioning in your `plugin.json`
3. **Create or use a marketplace**: Distribute through plugin marketplaces for easy installation
4. **Test with others**: Have team members test the plugin before wider distribution

## Related documentation

* [subagents.md](subagents.md) - Creating and using specialized agents
* [skills.md](skills.md) - Extend Claude's capabilities
* [hooks.md](hooks.md) - Event handling and automation
* [mcp.md](mcp.md) - External tool integration
* [commands.md](commands.md) - Custom slash commands
* [configuration.md](configuration.md) - Settings for plugins
