---
name: snowflake-mcp-claude-desktop
description: "Setup Snowflake managed MCP server with Claude Desktop using PAT authentication. Use when: connecting Claude Desktop to Snowflake MCP, setting up MCP server, configuring claude_desktop_config.json. Triggers: snowflake mcp, claude desktop mcp, mcp server setup, PAT authentication mcp."
---

# Snowflake Managed MCP + Claude Desktop Quickstart

## Prerequisites

- Snowflake account with CREATE MCP SERVER privilege
- Claude Desktop installed
- Node.js installed (`node --version` to verify)
- A warehouse for SQL execution

## Workflow

### Step 1: Create MCP Server in Snowflake

Execute in Snowflake:

```sql
-- Set context
USE ROLE ACCOUNTADMIN;  -- or role with CREATE MCP SERVER privilege
USE DATABASE <database>;
USE SCHEMA <schema>;

-- Create MCP server with SQL execution tool
CREATE OR REPLACE MCP SERVER <mcp_server_name>
  FROM SPECIFICATION $$
    tools:
      - name: "sql-exec"
        type: "SYSTEM_EXECUTE_SQL"
        description: "Execute SQL queries against Snowflake"
        title: "SQL Execution"
  $$;

-- Verify creation
SHOW MCP SERVERS;
DESCRIBE MCP SERVER <mcp_server_name>;
```

### Step 2: Create Role and Grant Permissions

```sql
-- Create dedicated role
CREATE ROLE IF NOT EXISTS <mcp_role>;

-- Grant MCP server usage
GRANT USAGE ON DATABASE <database> TO ROLE <mcp_role>;
GRANT USAGE ON SCHEMA <database>.<schema> TO ROLE <mcp_role>;
GRANT USAGE ON MCP SERVER <database>.<schema>.<mcp_server_name> TO ROLE <mcp_role>;

-- Grant warehouse
GRANT USAGE ON WAREHOUSE <warehouse> TO ROLE <mcp_role>;

-- Grant data access (adjust as needed)
GRANT SELECT ON ALL TABLES IN SCHEMA <database>.<schema> TO ROLE <mcp_role>;

-- Assign role to user
GRANT ROLE <mcp_role> TO USER <username>;
```

### Step 3: Create Programmatic Access Token (PAT)

**Option A - SQL:**
```sql
ALTER USER <username> ADD PROGRAMMATIC_ACCESS_TOKEN 
  NAME = 'claude_desktop_pat'
  COMMENT = 'PAT for Claude Desktop MCP connection'
  DEFAULT_SECONDARY_ROLES = ()
  DAYS_TO_EXPIRY = 90;
```

**Option B - Snowsight UI:**
1. Click username (bottom-left)
2. Select My Profile
3. Go to Programmatic Access Tokens
4. Click + Add
5. Name: `claude_desktop_pat`, set expiry
6. Copy token immediately (shown only once)

**STOP**: Save the PAT token securely before proceeding.

### Step 4: Configure Claude Desktop

Open config file:
- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

**IMPORTANT**: Claude Desktop requires `mcp-remote` bridge for remote MCP servers.

Add this configuration:

```json
{
  "mcpServers": {
    "snowflake": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://<account>.<region>.snowflakecomputing.com/api/v2/databases/<DATABASE>/schemas/<SCHEMA>/mcp-servers/<MCP_SERVER_NAME>",
        "--header",
        "Authorization: Bearer <YOUR_PAT_TOKEN>"
      ]
    }
  }
}
```

**Placeholders:**
| Placeholder | Example |
|-------------|---------|
| `<account>` | xy12345 |
| `<region>` | us-east-1 |
| `<DATABASE>` | MY_DATABASE |
| `<SCHEMA>` | MY_SCHEMA |
| `<MCP_SERVER_NAME>` | claude_mcp_server |
| `<YOUR_PAT_TOKEN>` | ver:1:abc123... |

### Step 5: Restart and Test

1. Fully quit Claude Desktop
2. Reopen Claude Desktop
3. Look for tools icon - Snowflake tools should appear

**Test prompts:**
- "Show me the tables in my_schema"
- "Run: SELECT CURRENT_USER(), CURRENT_ROLE(), CURRENT_WAREHOUSE()"

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Tools not appearing | Check JSON syntax, restart Claude Desktop |
| `command` field required error | You used URL format instead of mcp-remote format (see Step 4) |
| 401 Unauthorized | PAT expired or incorrect, regenerate |
| 403 Forbidden | Missing USAGE grant on MCP server |
| Connection refused | Check account URL format |
| npx not found | Install Node.js first |

## Stopping Points

- After Step 3: Save PAT token before proceeding
- After Step 4: Verify JSON syntax before restarting

## Output

Working Claude Desktop connected to Snowflake MCP server with SQL execution capability.
