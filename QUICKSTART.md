# Quick Start Guide

Get started building monday.com apps with Claude Code in minutes.

## Prerequisites

- Node.js 20+ (24 recommended)
- Docker (for local MongoDB)
- A monday.com developer account
- A monday.com API token (for MCP tools) - get it from Get your API token from https://<monday-slug>.monday.com/apps/manage/tokens

### MCP Token Setup

The `monday-code-init` and `monday-code-deploy` plugins use the monday-apps MCP server. Replace `${MONDAY_API_TOKEN}` in their `.mcp.json` files with your actual API token before installing.

## Installation

```bash
# Add the marketplace to Claude Code
/plugin marketplace add gregr/agentic-monday-apps-framework

# Install all plugins
/plugin install monday-code-init@agentic-monday-apps-framework
/plugin install monday-code-dev@agentic-monday-apps-framework
/plugin install monday-code-test@agentic-monday-apps-framework
/plugin install monday-code-deploy@agentic-monday-apps-framework
```

## Full Workflow: Build a Fullstack App

### 1. Create Your App on monday.com

Go to https://monday.com/developers/apps and create a new app. Note the **App ID** from the URL.

### 2. Initialize the Project

```bash
/monday-code-init fullstack
```

This creates:
- `frontend/` - React + Vite + TypeScript + @vibe/core + MondayContext
- `backend/` - Express + TypeScript + JWT auth + Document DB
- `.mondaycoderc` - Runtime configuration (Node.js 22)
- `manifest.template.json` - App manifest

### 3. Configure Environment

```bash
# Set your app ID
export MONDAY_APP_ID=<your-app-id>

# Backend .env (copy from .env.example)
MNDY_MONGODB_CONNECTION_STRING=mongodb://localhost:27017/my-app
MONDAY_CLIENT_SECRET=<from Developer Center > OAuth>
```

### 4. Start Development

```bash
/monday-dev
```

This starts:
- Local MongoDB via Docker on port 27017
- Backend on http://localhost:8080
- Frontend on http://localhost:3000

The frontend auto-detects local development and uses mock monday.com context.

### 5. Build Features

Your app comes with:
- **MondayContext** - React context with user info, theme, SDK access
- **JWT auth middleware** - Validates monday.com session tokens
- **Document DB connection** - MongoDB via `MNDY_MONGODB_CONNECTION_STRING`
- **Multi-tenant isolation** - All queries filtered by `accountId`

### 6. Write and Run Tests

```bash
/monday-test
```

Tests include JWT token generation matching monday.com's format and multi-tenant isolation patterns.

### 7. Deploy

```bash
/monday-deploy
```

This builds and pushes to monday-code (CDN for frontend, serverless for backend).

### 8. Post-Deploy Setup

After deploying:
- Set `MONDAY_CLIENT_SECRET` as an environment variable in monday-code
- Connect deployment to app features
- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected

## Key Patterns

### Monday SDK Context (Frontend)

```tsx
import { useMondayContext } from "./contexts/MondayContext";

function MyComponent() {
  const { monday, userId, accountId, theme, isLoading } = useMondayContext();
  // ...
}
```

### JWT Auth (Backend)

```typescript
// monday.com tokens: { dat: { user_id, account_id } }
// req.auth.userId and req.auth.accountId available on all /api routes
```

### Document DB

```typescript
import { getDb } from "./db/connection";

const db = await getDb();
const items = await db.collection("items")
  .find({ accountId: req.auth!.accountId })  // Always filter by tenant
  .toArray();
```

## Environment Variables

| Variable | Where | Description |
|----------|-------|-------------|
| `MONDAY_APP_ID` | Local shell | Your app's ID (for deploy commands) |
| `MONDAY_CLIENT_SECRET` | Backend .env + monday-code | JWT verification secret |
| `MNDY_MONGODB_CONNECTION_STRING` | Backend .env (local) / auto-injected (prod) | Document DB connection |
| `VITE_DEV_TOKEN` | Frontend .env (local only) | JWT for local API calls |
| `PORT` | Backend .env | Server port (default: 8080) |

## Troubleshooting

**"MNDY_MONGODB_CONNECTION_STRING not set"** - For local dev, start Docker MongoDB:
```bash
docker run -d --name monday-app-mongo -p 27017:27017 mongo:7
```

**"MONDAY_CLIENT_SECRET not configured"** - Find it in Developer Center > Your App > OAuth.

**Frontend shows "Loading..."** - In local dev, MondayContext uses mock data. If stuck, check browser console.

**Deploy fails** - Ensure `mapps` CLI is authenticated: `npx @mondaycom/apps-cli init`
