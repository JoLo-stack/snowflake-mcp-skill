# Snowflake MCP + Claude Desktop Quickstart

A Cortex Code skill for setting up Snowflake's managed MCP server with Claude Desktop using PAT authentication.

## What This Skill Does

Guides you through:
1. Creating an MCP server in Snowflake
2. Setting up roles and permissions
3. Generating a Programmatic Access Token (PAT)
4. Configuring Claude Desktop with `mcp-remote`
5. Testing the connection

## Key Insight

Claude Desktop doesn't natively support remote MCP servers via URL. You must use `mcp-remote` as a bridge:

```json
{
  "mcpServers": {
    "snowflake": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://<account>.snowflakecomputing.com/api/v2/databases/<DB>/schemas/<SCHEMA>/mcp-servers/<SERVER>",
        "--header",
        "Authorization: Bearer <PAT_TOKEN>"
      ]
    }
  }
}
```

## Prerequisites

- Snowflake account with CREATE MCP SERVER privilege
- Claude Desktop installed
- Node.js installed

## Usage in Cortex Code

Copy to your project:
```bash
mkdir -p .claude/skills/snowflake-mcp-claude-desktop
cp SKILL.md .claude/skills/snowflake-mcp-claude-desktop/
```

Then ask: *"Help me set up Snowflake MCP with Claude Desktop"*

## License

MIT
