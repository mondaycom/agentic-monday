# Monday Code Plugin

Build, deploy, and manage monday code apps on the monday.com platform.

## Installation

```bash
/plugin install monday-code@agentic-monday-apps-framework
```

## Skills

### `/monday-code-init` - Initialize a new app

Scaffold a new monday.com code app with proper structure, dependencies, and monday-code configuration.

```bash
/monday-code-init                # Interactive — asks for app type
/monday-code-init frontend       # React + Vite + monday-sdk-js
/monday-code-init backend        # Express + @mondaycom/apps-sdk
/monday-code-init fullstack      # Frontend + backend
```

**What it does:**
1. Creates `.mondaycoderc` (Node.js runtime selection) and `manifest.json` (app features)
2. Scaffolds frontend with Vite, React, MondayContext provider, and API service layer
3. Scaffolds backend with Express, JWT auth middleware, MongoDB connection, secrets manager, and queue utilities
4. Sets up multi-tenant data isolation pattern (accountId filtering)
5. Creates `.env.example` files and installs dependencies
6. Suggests running `/monday-code-dev` as next step

### `/monday-code-migrate` - Migrate an existing app

Migrate an existing frontend, backend, or fullstack app to the monday-code platform structure.

```bash
/monday-code-migrate              # Interactive — analyzes project first
/monday-code-migrate frontend     # Migrate frontend only
/monday-code-migrate backend      # Migrate backend only
/monday-code-migrate fullstack    # Migrate both
```

**What it does:**
1. Analyzes existing project (framework, build tool, language, package manager, structure)
2. Presents a migration plan for user approval before making changes
3. Adds monday-code configuration (`.mondaycoderc`, `manifest.json`)
4. Integrates monday SDK (MondayContext, session tokens, API service layer)
5. Adds monday-code entry points (`index.js`, `preload.cjs`), auth middleware, and SDK utilities
6. Handles database migration guidance (MongoDB/Document DB) and multi-tenant isolation
7. Preserves existing business logic — only adds monday-code compatibility
8. Runs build verification and presents a migration checklist

### `/monday-code-dev` - Start development servers

Start development servers and set up the local environment for monday code apps.

```bash
/monday-code-dev                  # Auto-detect and start all servers
/monday-code-dev frontend         # Start frontend dev server only
/monday-code-dev backend          # Start backend dev server only
/monday-code-dev all              # Start both
```

**What it does:**
1. Detects project structure (frontend, backend, or fullstack)
2. Runs pre-flight checks (dependencies installed, `.env` files present)
3. Starts local MongoDB via Docker if the backend uses Document DB
4. Starts dev servers (Vite on port 3000, Express with tsx watch on port 8080)
5. Supports local dev mode with mock monday SDK data (no monday.com connection needed)
6. Provides tunnel setup instructions (`mapps tunnel:create`) for testing with real monday.com webhooks

### `/monday-code-deploy` - Build, deploy, and manage

Build and deploy monday code apps to the monday-code platform (serverless + CDN).

```bash
/monday-code-deploy               # Build and deploy (auto-detect type)
/monday-code-deploy frontend      # Deploy frontend to CDN only
/monday-code-deploy backend       # Deploy backend to serverless only
/monday-code-deploy status        # Check deployment status
/monday-code-deploy env           # Manage environment variables
/monday-code-deploy cron          # Set up cron jobs
/monday-code-deploy alerts        # Configure monitoring alerts
```

**What it does:**
1. Pre-flight checks (`MONDAY_APP_ID`, `mapps` CLI auth, build verification)
2. Builds and pushes to monday-code (CDN for frontend, serverless for backend)
3. Connects deployments to app features
4. Manages environment variables and secrets

**Deployment types:**

| App Type | Destination | Flag |
|----------|-------------|------|
| Frontend (React/Vite) | CDN | `-c` |
| Backend (Express) | Serverless | (none) |
| Fullstack | Both | Sequential |

**Advanced features:**
- **Security scanning** — `mapps code:push -s` scans deployment artifacts
- **Multi-region** — Deploy to US, EU, AU, IL regions
- **Cron jobs** — Schedule recurring tasks via `/mndy-cronjob/` routes (max 5/region)
- **Alerts** — HTTP error rate, latency, and runtime quota monitoring
- **Secrets Manager** — Runtime access to sensitive values via `@mondaycom/apps-sdk`
- **App promotion** — Promote development versions to live

## Key Notes

- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected by monday-code after first deploy — never set manually
- Enable multi-region BEFORE using Document DB (irreversible)
- Cron routes must use `/mndy-cronjob/` prefix and be POST handlers
- All database queries must filter by `accountId` for multi-tenant isolation

## Requirements

- `MONDAY_APP_ID` environment variable
- `@mondaycom/apps-cli` authenticated (`mapps init`)
- App created in monday.com Developer Center
