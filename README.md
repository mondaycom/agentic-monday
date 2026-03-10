# Monday Apps Framework

A Claude Code plugin marketplace for the full SDLC of monday.com apps.

## Prerequisites

The `monday-code` plugin uses the monday-apps MCP server. Before installing, set your monday.com API token in the `.mcp.json` file in the plugin:

1. Get your API token from https://<monday-slug>.monday.com/apps/manage/tokens
2. Replace `${MONDAY_API_TOKEN}` in the `.mcp.json` files under `plugins-official/monday-code`

## Quick Start

```bash
# Add the marketplace
/plugin marketplace add mondaycom/agentic-monday-apps-framework

# Install plugins
/plugin install monday-code@agentic-monday-apps-framework

# Use a skill
/monday-code-init fullstack
```

## Available Plugins

| Plugin | Skill | Description |
|--------|-------|-------------|
| [monday-code](./plugins-official/monday-code/) | `/monday-code-init` | Scaffold frontend/backend/fullstack apps with monday SDK, JWT auth, Document DB, and multi-tenant patterns |
| [monday-code](./plugins-official/monday-code/) | `/monday-code-migrate` | Migrate existing apps to monday-code — build-tool agnostic, preserves existing code |
| [monday-code](./plugins-official/monday-code/) | `/monday-code-dev` | Start dev servers, local MongoDB, tunnel setup |
| [monday-code](./plugins-official/monday-code/) | `/monday-code-deploy` | Deploy to monday-code with multi-region, cron, alerts, security scanning |

## What You Get

Apps scaffolded by this framework include:

- **Monday SDK integration** - MondayContext provider with local dev detection
- **JWT authentication** - Middleware supporting both production and dev token formats
- **Document DB** - monday code MongoDB connection with `MNDY_MONGODB_CONNECTION_STRING`
- **Vibe Design System** - `@vibe/core` for monday.com-native UI
- **TypeScript** - End-to-end type safety
- **monday-code ready** - `.mondaycoderc`, proper entry points, deploy scripts

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for plugin development guidelines.

## License

MIT
