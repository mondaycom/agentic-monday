# Agentic monday.com

A Claude Code plugin marketplace for the building on monday.com

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

| Plugin | Skills | Description |
|--------|--------|-------------|
| [monday-code](./plugins-official/monday-code/) | `/monday-code-init`, `/monday-code-migrate`, `/monday-code-dev`, `/monday-code-deploy`, `/monday-code-ops` | Build, deploy, and manage monday code apps on the monday.com platform |

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
