# Monday Code Dev

Start development servers and local environment for monday code apps.

## Installation

```bash
/plugin install monday-code-dev@agentic-monday-apps-framework
```

## Usage

```bash
/monday-dev              # Auto-detect and start all servers
/monday-dev frontend     # Frontend only
/monday-dev backend      # Backend only
```

## What It Does

1. Detects project type (frontend/backend/fullstack)
2. Checks and installs dependencies
3. Starts local MongoDB via Docker (if backend uses Document DB)
4. Starts dev servers with hot reload

## Development URLs

| Service | Port | URL |
|---------|------|-----|
| Frontend (Vite) | 3000 | http://localhost:3000 |
| Backend (Express) | 8080 | http://localhost:8080 |
| MongoDB | 27017 | mongodb://localhost:27017 |
| Health check | 8080 | http://localhost:8080/health |

## Local Dev Mode

The frontend MondayContext auto-detects local development (not inside monday.com iframe) and provides mock user data. No monday.com connection needed for UI development.

For testing with real monday.com webhooks, use a tunnel:
```bash
ngrok http 8080
```

## Requirements

- Node.js 18+
- Docker (for MongoDB)
- `.env` configured (copy from `.env.example`)
