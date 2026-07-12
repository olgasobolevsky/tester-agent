This repository uses a Jira MCP server for AI-assisted Jira access.

## Jira MCP

- VS Code MCP client configuration lives in `.vscode/mcp.json`.
- The Jira MCP server is started with Docker from `setup/docker/jira-mcp/docker-compose.yml`.
- Jira credentials are provided through a local `.env` file. Use `.env.example` as the template.
- If Jira tools are unavailable in chat, assume the MCP server is not running or not connected.

## Expected Agent Behavior

- When a user provides a Jira ticket key, read the ticket before creating a plan or implementing tests.
- Before writing back to Jira, confirm with the user.
- Prefer concise comments or summaries over broad field updates unless the user explicitly requests a ticket mutation.
- Do not commit or push changes to git unless explicitly asked by the user.