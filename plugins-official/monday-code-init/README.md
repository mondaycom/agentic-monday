# Monday Code Init

Scaffold new monday.com code apps with production-ready patterns.

## Installation

```bash
/plugin install monday-code-init@agentic-monday-apps-framework
```

## Usage

```bash
/monday-code-init              # Interactive - asks for app type
/monday-code-init fullstack    # Scaffold frontend + backend
/monday-code-init frontend     # Frontend only
/monday-code-init backend      # Backend only
```

## What It Creates

### Frontend
- React 18 + TypeScript + Vite
- `@vibe/core` (monday.com design system)
- `MondayContext` provider with local dev detection
- API service with JWT session token auth
- monday-code CDN deploy script

### Backend
- Express + TypeScript
- JWT auth middleware (supports production + dev token formats)
- Document DB connection (`MNDY_MONGODB_CONNECTION_STRING`)
- Secrets Manager integration (`@mondaycom/apps-sdk`)
- monday-code serverless deploy script

### Configuration
- `.mondaycoderc` - Node.js 22 runtime
- `manifest.template.json` - App manifest with feature type
- `.env.example` - Required environment variables
- Multi-tenant data patterns (accountId isolation)

## Key Patterns

- **MondayContext** - Auto-detects local dev vs monday.com iframe
- **JWT auth** - Handles both `{ dat: { user_id } }` (prod) and `{ userId }` (dev) formats
- **Document DB** - Uses `MNDY_MONGODB_CONNECTION_STRING` (auto-injected in prod)
- **Multi-tenant** - All DB queries filtered by `accountId`

## Requirements

- Node.js 18+
- Docker (for local MongoDB)
- monday.com developer account
