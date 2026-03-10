# Monday Code Deploy

Build, deploy, and manage monday code apps on the monday-code platform.

## Installation

```bash
/plugin install monday-code-deploy@agentic-monday-apps-framework
```

## Usage

```bash
/monday-deploy           # Build and deploy (auto-detect type)
/monday-deploy frontend  # Deploy frontend to CDN only
/monday-deploy backend   # Deploy backend to serverless only
/monday-deploy status    # Check deployment status
/monday-deploy env       # Manage environment variables
/monday-deploy cron      # Set up cron jobs
/monday-deploy alerts    # Configure monitoring alerts
```

## What It Does

1. Pre-flight checks (MONDAY_APP_ID, build, auth)
2. Builds and pushes to monday-code (CDN for frontend, serverless for backend)
3. Connects deployments to app features
4. Manages environment variables and secrets

## Deployment Types

| App Type | Destination | Flag |
|----------|-------------|------|
| Frontend (React/Vite) | CDN | `-c` |
| Backend (Express) | Serverless | (none) |
| Fullstack | Both | Sequential |

## Advanced Features

- **Security scanning** - `mapps code:push -s` scans deployment artifacts
- **Multi-region** - Deploy to US, EU, AU, IL regions
- **Cron jobs** - Schedule recurring tasks via `/mndy-cronjob/` routes (max 5/region)
- **Alerts** - HTTP error rate, latency, and runtime quota monitoring
- **Secrets Manager** - Runtime access to sensitive values via `@mondaycom/apps-sdk`

## Key Notes

- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected by monday-code - never set manually
- Enable multi-region BEFORE using Document DB (irreversible)
- Cron routes must use `/mndy-cronjob/` prefix and be POST handlers

## Requirements

- `MONDAY_APP_ID` environment variable
- `@mondaycom/apps-cli` authenticated (`mapps init`)
- App created in monday.com Developer Center
