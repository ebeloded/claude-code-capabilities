# Connect Claude Code to tools via MCP

> Learn how to connect Claude Code to your tools with the Model Context Protocol.

The Model Context Protocol (MCP) allows Claude Code to connect to external tools and services. MCP servers can provide:

* **Tools**: Functions that Claude can call
* **Resources**: Data that Claude can read
* **Prompts**: Reusable prompt templates

## Quick Start

### Add an MCP server

Use the `/mcp` command to interactively configure MCP servers:

```
/mcp
```

Or use the CLI:

```bash
# HTTP server
claude mcp add --transport http server-name https://mcp.example.com/mcp

# SSE server
claude mcp add --transport sse server-name https://mcp.example.com/sse

# STDIO server (local)
claude mcp add --transport stdio server-name -- npx -y @package/mcp-server

# With environment variables
claude mcp add --transport stdio server-name --env API_KEY=YOUR_KEY -- npx -y server
```

### Common MCP servers

Popular MCP servers you can connect to:

**Project Management**
* **Asana**: `https://mcp.asana.com/sse`
* **Atlassian** (Jira, Confluence): `https://mcp.atlassian.com/v1/sse`
* **Linear**: `https://mcp.linear.app/mcp`
* **Notion**: `https://mcp.notion.com/mcp`

**Development Tools**
* **Figma**: `https://mcp.figma.com/mcp`
* **Sentry**: `https://mcp.sentry.dev/mcp`
* **GitHub**: Available via npm packages

**Data & Analytics**
* **Airtable**: `npx -y airtable-mcp-server`

**Payments & Commerce**
* **Stripe**: `https://mcp.stripe.com`
* **PayPal**: `https://mcp.paypal.com/mcp`
* **Square**: `https://mcp.squareup.com/sse`

## MCP Server Types

### Remote servers (HTTP/SSE)

Remote MCP servers are hosted externally and accessed over the internet.

* **HTTP**: Request/response pattern
* **SSE**: Server-sent events for streaming

**Example:**
```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

### Local servers (STDIO)

Local MCP servers run on your machine and communicate via standard input/output.

**Example:**
```bash
claude mcp add --transport stdio github -- npx -y @modelcontextprotocol/server-github
```

## Configuration

### MCP configuration files

MCP servers can be configured in multiple locations:

* **User level**: `~/.claude/mcp.json` - Available across all projects
* **Project level**: `.mcp.json` - Shared with your team
* **Enterprise level**: Managed MCP configuration

### Project .mcp.json format

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/files"]
    }
  }
}
```

### Enabling project MCP servers

When you open a project with a `.mcp.json` file, Claude Code will ask for permission to enable the servers.

You can also configure automatic approval in `.claude/settings.json`:

```json
{
  "enableAllProjectMcpServers": true
}
```

Or approve specific servers:

```json
{
  "enabledMcpjsonServers": ["github", "filesystem"]
}
```

## Managing MCP Servers

### View connected servers

```
/mcp
```

Select "View configured servers" to see all connected MCP servers and their status.

### Test a server

After adding an MCP server, test it by asking Claude to use one of its tools:

```
> Use the GitHub MCP server to list my repositories
> Search for issues using the Atlassian MCP server
```

### Remove a server

Use the `/mcp` command and select "Remove server", or manually edit your MCP configuration files.

## Permissions

### MCP tool permissions

Control which MCP tools Claude can use with permissions in `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "mcp__github",                    // Allow all tools from github server
      "mcp__github__get_issue"          // Allow specific tool
    ],
    "deny": [
      "mcp__filesystem"                 // Block all tools from filesystem server
    ]
  }
}
```

**Note**: MCP permissions do NOT support wildcards. To approve all tools from a server, use the server name: `mcp__github` (not `mcp__github__*`).

### Subagents and MCP tools

Subagents can access MCP tools. When the `tools` field is omitted in a subagent definition, it inherits all MCP tools available to the main thread.

See [subagents.md](subagents.md) for details.

## Enterprise MCP Configuration

Enterprises can deploy managed MCP servers and policies:

* **Managed MCP servers**: Deployed at system level, cannot be overridden
* **Allowlist/Denylist**: Control which MCP servers users can configure
* **Lockdown mode**: Restrict to only enterprise-managed servers

### Managed MCP configuration files

* macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
* Linux/WSL: `/etc/claude-code/managed-mcp.json`
* Windows: `C:\ProgramData\ClaudeCode\managed-mcp.json`

### Enterprise policy settings

In `managed-settings.json`:

```json
{
  "useEnterpriseMcpConfigOnly": true,  // Only use managed MCP servers
  "allowedMcpServers": [                // Allowlist
    { "serverName": "github" }
  ],
  "deniedMcpServers": [                 // Denylist (takes precedence)
    { "serverName": "filesystem" }
  ]
}
```

## Troubleshooting

### Server not connecting

1. Check the server URL or command is correct
2. Verify environment variables are set
3. Check network connectivity for remote servers
4. Look for errors in Claude Code logs (`--verbose` mode)

### Tools not appearing

1. Restart Claude Code after adding a server
2. Check permissions are not blocking MCP tools
3. Verify the server is running (for local servers)

### Authentication issues

For OAuth-based servers:
1. Use `/mcp` to manage authentication
2. Follow the OAuth flow to authorize
3. Tokens are stored securely in your system keychain

## Related documentation

* [permissions.md](permissions.md) - Control MCP tool access
* [configuration.md](configuration.md) - MCP configuration settings
* [plugins.md](plugins.md) - Plugins can include MCP servers
* [subagents.md](subagents.md) - Subagents can use MCP tools
