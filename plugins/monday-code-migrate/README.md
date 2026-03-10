# Monday Code Migrate

Migrate existing frontend, backend, or fullstack apps to the monday-code platform.

## Installation

```bash
/plugin install monday-code-migrate@agentic-monday-apps-framework
```

## Usage

```bash
/monday-code-migrate:monday-migrate              # Interactive - asks for migration scope
/monday-code-migrate:monday-migrate fullstack     # Migrate frontend + backend
/monday-code-migrate:monday-migrate frontend      # Frontend only
/monday-code-migrate:monday-migrate backend        # Backend only
```

## What It Does

### Analysis
- Detects existing framework, build tool, language, and project structure
- Identifies gaps vs monday-code requirements
- Presents a migration plan for approval before making changes

### Frontend Migration
- Adds `monday-sdk-js` and `MondayContext` provider
- Configures `global = globalThis` for the existing build tool (Vite, Webpack, esbuild, Parcel, etc.)
- Creates API service layer with monday session token auth
- Adds `index.js` CDN entry point and deploy script
- Works with any build tool that outputs static files — no build tool changes required

### Backend Migration
- Splits app/server entry points for monday-code serverless
- Adds JWT auth middleware (supports production + dev token formats)
- Adds automation/webhook verification middleware
- Integrates `@mondaycom/apps-sdk` (secrets, env vars, logger)
- Document DB connection with `MNDY_MONGODB_CONNECTION_STRING`
- Migrates CommonJS to ESM if needed
- Flags database queries missing multi-tenant `accountId` filtering

### Configuration
- `.mondaycoderc` - Node.js 22 runtime
- `manifest.json` - App manifest with correct feature type schema
- `.env.example` - Required environment variables
- Multi-tenant data patterns (accountId isolation)

## Key Principles

- **Preserve existing code** - Migration adds monday-code compatibility, never rewrites business logic
- **Build-tool agnostic** - Works with Vite, Webpack, esbuild, Parcel, CRA, or any static build output
- **Plan before act** - Always presents changes for user approval before modifying files
- **Minimal changes** - Only adds what's necessary for monday-code compatibility

## Requirements

- Node.js 20+
- Existing frontend (React, Vue, Angular, vanilla JS) and/or backend (Express, Fastify, Koa, Hono, NestJS)
- monday.com developer account
