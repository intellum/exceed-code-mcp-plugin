# Exceed Admin API — Claude Code Plugin

Give Claude access to the Exceed admin API through two code-mode MCP tools: **search** (discover endpoints) and **execute** (make API calls).

## Install

```
/plugin add github:intellum/exceed-code-mcp-plugin
```

Claude Code will ask for your **Exceed domain URL** (e.g., `https://mycompany.exceedlms.com`).

## Login

```
/exceed-login
```

Opens your browser to Exceed's OAuth authorization page. Log in, authorize, copy the code, paste it back. Token is saved for the session.

## Use

After login, restart Claude Code. You'll have two tools:

### search
Query the Exceed admin API specification (~260 endpoints):
- "What endpoints are available for managing users?"
- "Show me the request schema for creating a course"
- "Find all enrollment-related endpoints"

### execute
Make authenticated API calls:
- "List all users in my account"
- "Show enrollments for course 42"
- "Create a new letter template"

## Safety

- Bulk destructive operations (`bulk_destroy`, `bulk_delete`) are blocked
- Rate limited: 10 req/s global, 5 req/s writes
- Max 25 requests per execution, max 5 DELETEs
- Regular admins can't access super_admin-only endpoints (server-side role check)

## Commands

| Command | Description |
|---------|-------------|
| `/exceed-login` | Authenticate with your Exceed instance |
| `/exceed-setup` | Configure which Exceed instance to connect to |

## Requirements

- An Exceed instance with the `exceed-code-mcp` OAuth app registered
- Admin access on the Exceed instance

## How it works

This plugin connects to a hosted MCP server that proxies the Exceed admin API. Your OAuth token is sent with each request — no credentials are stored on the server.

```
Claude Code → Plugin → Hosted MCP Server → Exceed Admin API
                         (Cloud Run)
```

The MCP server loads the Exceed OpenAPI spec at startup and exposes it through the `search` tool. The `execute` tool forwards authenticated requests to your Exceed instance.
