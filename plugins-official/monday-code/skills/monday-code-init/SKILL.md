---
name: monday-code-init
description: "Scaffolds a new monday.com code app with the correct structure, dependencies, and monday-code deployment configuration. Use when user wants to 'create a monday code app', 'scaffold a new app', 'initialize a monday app', 'set up a frontend app', 'set up a backend app', 'create a fullstack monday app', or asks how to get started building on monday-code."
argument-hint: "[frontend|backend|fullstack]"
user-invocable: true
allowed-tools: ["Bash", "Write", "Read", "Glob", "Grep", "mcp__monday-apps__*"]
---

# monday-code-init

Initialize a new monday.com code app with the correct structure for monday-code deployment.

## When to Use

- User wants to create a new monday code app
- User wants to scaffold a frontend-only app (React + Vite + monday-sdk-js)
- User wants to scaffold a backend-only app (Express + @mondaycom/apps-sdk)
- User wants to scaffold a fullstack app (frontend + backend)

## Usage Examples

```
/monday-code-init fullstack
/monday-code-init frontend
/monday-code-init backend
```

Or conversationally: "Create a new monday code app", "Scaffold a fullstack monday app for me", "Set up a backend app with queue support".

## Input Gathering

Ask the user:
1. **App type**: frontend | backend | fullstack
2. **App name**: Used for directory names and package.json
3. **App feature type**: What monday.com feature they're building (BoardView, ItemView, DashboardWidget, Integration, etc.)

If the user provided an argument (e.g., `/monday-code-init fullstack`), skip the app type question.

## Instructions

> Full code templates for all files below are in [references/templates.md](references/templates.md). Copy them verbatim, substituting `{{APP_NAME}}` and `{{FEATURE_NAME}}`.

### Step 1: Create .mondaycoderc

Create a `.mondaycoderc` file in the project root to specify the Node.js runtime:

```json
{
  "runtime": "nodejs22.x"
}
```

Supported runtimes: `nodejs20.x`, `nodejs22.x`, `nodejs24.x`. Default to `nodejs22.x`.

### Step 2: Create manifest.json

Create the monday app manifest in the project root:

```json
{
  "name": "{{APP_NAME}}",
  "description": "A monday.com app",
  "features": [
    {
      "type": "AppFeatureProductView",
      "name": "{{FEATURE_NAME}}",
      "settings": {
        "hideViewHeader": true,
        "isMobileSupported": false
      }
    }
  ],
  "oauthScopes": []
}
```

Use the apps mcp get_app_feature_schema tool to fetch the correct schema for the user's chosen feature type and adjust the `settings` accordingly.

Adjust `features[0].type` based on the user's feature type choice. Common types:
- `AppFeatureProductView` - Full-page view
- `AppFeatureBoardView` - Board view
- `AppFeatureItemView` - Item view
- `AppFeatureDashboardWidget` - Dashboard widget

### Step 3: Frontend (if frontend or fullstack)

Create the following structure:

```
frontend/
├── package.json
├── tsconfig.json
├── vite.config.ts
├── index.html
├── index.js
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── contexts/
    │   └── MondayContext.tsx
    ├── services/
    │   └── api.ts
    └── types/
        └── index.ts
```

See [references/templates.md](references/templates.md) for full file contents:
- `frontend/package.json` — React + Vite + monday-sdk-js + @vibe/core
- `frontend/index.html` — HTML entry point with root div and module script tag
- `frontend/vite.config.ts` — Vite config with `global: "globalThis"` polyfill
- `frontend/index.js` — CDN entry point (required by monday-code)
- `frontend/src/contexts/MondayContext.tsx` — **Critical**: monday SDK integration with local dev mock support
- `frontend/src/services/api.ts` — Backend API helper using session tokens and mondayCodeHostingUrl
- `frontend/src/App.tsx` — Root component using MondayContext
- `frontend/src/main.tsx` — React entry point wrapping app in MondayProvider
- `frontend/.env.example` — VITE_DEV_TOKEN for local development

### Step 4: Backend (if backend only or fullstack)

Create the following structure:

```
backend/
├── package.json
├── tsconfig.json
├── index.js
├── preload.cjs
└── src/
    ├── server.ts
    ├── app.ts
    ├── routes/
    │   └── health.ts
    ├── middleware/
    │   └── auth.ts
    ├── db/
    │   └── connection.ts
    ├── types/
    │   └── index.ts
    └── utils/
        ├── logger.ts
        ├── secrets.ts
        ├── env-vars.ts
        ├── storage.ts (optional, for object/BLOB storage with @mondaycom/apps-sdk >= 3.3.1)
        └── queue.ts (optional, for async processing with monday-code queues)
```

See [references/templates.md](references/templates.md) for full file contents:
- `backend/package.json` — Express + @mondaycom/apps-sdk + mongodb + jsonwebtoken
- `backend/preload.cjs` — dotenv loader for local dev
- `backend/index.js` — monday-code serverless entry point
- `backend/src/app.ts` — Express app with CORS, JSON parsing, auth middleware
- `backend/src/routes/health.ts` — Health check endpoint
- `backend/src/middleware/auth.ts` — JWT auth (two variants: session tokens and automation webhooks)
- `backend/src/db/connection.ts` — MongoDB connection using MNDY_MONGODB_CONNECTION_STRING
- `backend/src/utils/secrets.ts` — SecretsManager with local dev fallback
- `backend/src/utils/env-vars.ts` — EnvironmentVariablesManager wrapper
- `backend/src/utils/storage.ts` — Optional: Object storage (BLOB) for files, images, documents (requires `@mondaycom/apps-sdk` >= 3.3.1)
- `backend/src/utils/queue.ts` — Optional: Queue publish/consume with secret validation
- `backend/.env.example` — MNDY_MONGODB_CONNECTION_STRING, MONDAY_CLIENT_SECRET

#### Auth Middleware Selection

- **Fullstack / session tokens** → use the `authMiddleware` variant (extracts `userId`/`accountId` from JWT `dat` field or direct fields)
- **Automation webhook** → use the `authenticationMiddleware` variant (validates MONDAY_SIGNING_SECRET, exposes `shortLivedToken` for GraphQL calls back to monday.com)

#### Document DB Notes

- Connection string env var: `MNDY_MONGODB_CONNECTION_STRING` (auto-injected by monday-code after first deploy)
- Max storage: 1 GiB per app
- Limits: 50,000 reads / 20,000 writes per region per day
- Requires `@mondaycom/apps-cli` >= 4.10.2
- Data is segregated per account automatically

#### Object Storage (Optional)

Include `backend/src/utils/storage.ts` when the user needs to store or serve files (images, documents, videos, archives). Object storage is backed by Google Cloud Storage and is designed for large, unstructured files with infrequent read/write operations.

See [references/templates.md](references/templates.md) for the full `storage.ts` template.

**When to include:**
- User mentions file uploads, image storage, document management, or BLOB storage
- User asks about storing large binary data
- User needs presigned URLs for direct client uploads

**Key constraints:**
- Requires `@mondaycom/apps-sdk` >= 3.3.1
- Presigned uploads: max 50 MB per file
- Presigned URL expiration: default 15 min, max 7 days
- Server-side uploads: no SDK-enforced limit (practical limits depend on runtime)
- Only available from backend code running on monday code (not local dev)

### Step 5: Environment variables

Create `.env.example` files (see [references/templates.md](references/templates.md) for full content):
- `backend/.env.example` — MNDY_MONGODB_CONNECTION_STRING, MONDAY_CLIENT_SECRET, PORT
- `frontend/.env.example` — VITE_DEV_TOKEN (local dev only)

### Step 6: Multi-Tenant Data Pattern

**CRITICAL**: All database queries MUST filter by `accountId` for multi-tenant isolation. Every document should include:

```typescript
interface BaseDocument {
  accountId: string;   // Multi-tenant isolation (from JWT)
  ownerId: string;     // Per-user access control (from JWT)
  createdAt: string;
  updatedAt: string;
}
```

When querying, always include `accountId`:
```typescript
// CORRECT - filtered by tenant
const items = await collection.find({ accountId: req.auth!.accountId }).toArray();

// WRONG - exposes all tenants' data
const items = await collection.find({}).toArray();
```

### Step 7: Post-Initialization

After creating all files:

1. Run `npm install` in each directory
2. Copy `.env.example` to `.env` and fill in values
3. Tell the user to:
   - Create an app at https://<slug>.monday.com/developers/apps
4. Important! You must suggest next step: `/monday-dev` to start development

### Step 8: Use MCP Tools

If the monday-apps MCP is configured (requires `MONDAY_API_TOKEN` in `.mcp.json`), use these tools:
- `mcp__monday-apps__monday_apps_get_development_context` - Get reference patterns
- `mcp__monday-apps__monday_apps_create_app` - Create the app programmatically
- `mcp__monday-apps__monday_apps_create_app_feature` - Create features
- `mcp__monday-apps__monday_apps_get_app_feature_schema` - Get feature schemas

## Error Handling

- If `npm install` fails: check Node.js version matches `.mondaycoderc` runtime (`nodejs22.x` = Node 22)
- If `MNDY_MONGODB_CONNECTION_STRING` is missing locally: run `docker run -d -p 27017:27017 mongo:7` and set the env var
- If JWT verification fails: confirm `MONDAY_CLIENT_SECRET` matches the app's signing secret in monday.com developer center
- If `mondayCodeHostingUrl` is undefined in frontend: the app must be deployed to monday-code at least once
- If auth middleware returns 401: ensure the frontend sends `Authorization: <sessionToken>` (not `Bearer <token>`) to match the backend's parsing logic

## Notes

- Always use TypeScript
- Use `MNDY_MONGODB_CONNECTION_STRING` for database connection (auto-injected by monday-code after first deploy)
- Optionally include `.mondaycoderc` for runtime selection
- Add multi-tenant isolation (accountId) to all DB queries
- Use the MondayContext pattern for frontend SDK integration
- Support both production and local dev JWT token formats in auth middleware
