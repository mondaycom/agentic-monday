---
name: monday-code-migrate
description: "Use when user says 'migrate my app to monday-code', 'convert to monday platform', 'add monday SDK to existing app', 'deploy existing app on monday', 'move my React/Node app to monday-code', or wants to add monday.com integration to an existing frontend, backend, or fullstack app. Migrates the existing project to monday-code platform structure while preserving existing code."
argument-hint: "[frontend|backend|fullstack]"
user-invocable: true
---

# monday-code-migrate

Migrate an existing app to the monday-code platform. This skill analyzes the current project, identifies what's missing, and adds the required monday-code structure while preserving existing code.

## When to Use

- User has an existing React frontend they want to run on monday-code CDN
- User has an existing Express/Node.js backend they want to run on monday-code serverless
- User has a fullstack app they want to migrate to monday-code
- User wants to add monday SDK integration to an existing app
- User wants to convert a standalone app into a monday.com app

## Usage Example

**User:** "I have an existing React + Express app I want to run on monday-code. How do I migrate it?"

**Actions:**
1. Detect existing structure: React frontend with Vite, Express backend, TypeScript
2. Present migration plan: add `.mondaycoderc`, `manifest.json`, `MondayContext`, auth middleware, secrets utils
3. User approves plan
4. Execute migration steps, preserving all existing business logic
5. Run `npm run build` in both frontend and backend to verify

**Result:** App is monday-code compatible with CDN-served frontend and serverless backend, ready to deploy with `/monday-code-deploy`.

## Input Gathering

Ask the user:
1. **Migration scope**: frontend | backend | fullstack (skip if provided as argument)
2. **App feature type**: What monday.com feature they're building (BoardView, ItemView, DashboardWidget, etc.)
3. **Existing framework**: Auto-detect from package.json, confirm with user

## Instructions

### Step 1: Analyze Existing Project

Scan the project to understand its current structure:

```
# Check for existing files
- package.json (root, frontend/, backend/, src/, etc.)
- tsconfig.json
- vite.config.ts / next.config.js / webpack.config.js
- src/ directory structure
- .env files
- Any existing monday SDK usage
```

**Detect the current setup:**
- **Frontend framework**: React, Vue, Angular, vanilla JS
- **Build tool**: Vite, Webpack, Next.js, CRA
- **Backend framework**: Express, Fastify, Koa, Hono, NestJS
- **Language**: TypeScript or JavaScript
- **Package manager**: npm, yarn, pnpm
- **Monorepo structure**: Is it already split into frontend/backend directories?

Report findings to the user before making changes.

### Step 2: Plan Migration

Based on the analysis, create a migration plan. Present this to the user for approval before proceeding.

The plan should list:
- Files to **create** (new monday-code files)
- Files to **modify** (existing files that need changes)
- Files to **move** (if restructuring is needed)
- Dependencies to **add**
- Dependencies to **remove** (if replacing, e.g., removing next.js server-side features)

**IMPORTANT**: Always preserve existing application logic. Migration adds monday-code compatibility around existing code — it does NOT rewrite business logic. Migration changes should be kept to a minimum necessary to achieve compatibility. Do only specific monday code changes, do NOT refactor the existing codebase (no JS to TS, no Fastify to Express, etc.) unless absolutely required for compatibility. Then you MUST first accept the consent of the user before making any such changes.

### Step 3: Root Configuration

Create or update these files in the project root (if they don't already exist):

**.mondaycoderc:**
```json
{
  "runtime": "nodejs22.x"
}
```

**manifest.json** — Use `mcp__monday-apps__monday_apps_get_app_feature_schema` to fetch the correct schema for the user's feature type and create the manifest file (`manifest.json`) with the appropriate content:
```json
{
  "name": "{{APP_NAME}}",
  "description": "A monday.com app",
  "features": [
    {
      "type": "{{FEATURE_TYPE}}",
      "name": "{{FEATURE_NAME}}",
      "settings": {}
    }
  ],
  "oauthScopes": []
}
```

### Step 4: Frontend Migration (if frontend or fullstack)

#### 4.1: Directory Structure

If the project is NOT already split into `frontend/` and `backend/` directories:
- Ask the user if they want to restructure into `frontend/` and `backend/` directories
- If yes, move existing frontend files into `frontend/`
- If no, work with the existing structure (adjust paths accordingly in all subsequent steps)

#### 4.2: Build Tool Configuration

monday-code CDN serves static files from a `dist/` directory. Any build tool that outputs static HTML/JS/CSS to `dist/` will work (Vite, Webpack, esbuild, Parcel, Rollup, etc.). Keep the existing build tool — do NOT switch build tools unless the user requests it.

**Required for monday-sdk-js**: The `global` variable must be defined. How to do this depends on the build tool:

- **Vite** — add `define: { global: "globalThis" }` to `vite.config.ts`
- **Webpack** — add `new webpack.ProvidePlugin({ global: "globalThis" })` to plugins, or add `node: { global: true }` to config
- **esbuild** — add `--define:global=globalThis` flag or `define: { global: "globalThis" }` in build config
- **Parcel** — typically handles this automatically; if not, add `<script>var global = globalThis;</script>` to `index.html` before the app script
- **No build tool / vanilla** — add `<script>var global = globalThis;</script>` to `index.html`

#### 4.3: monday SDK Integration

**Add monday-sdk-js:**
```bash
npm install monday-sdk-js
```

**Create `src/contexts/MondayContext.tsx`** (if it doesn't exist):
```tsx
import React, { createContext, useContext, useEffect, useState } from "react";
import mondaySdk from "monday-sdk-js";

const monday = mondaySdk();

interface MondayContextType {
  monday: ReturnType<typeof mondaySdk>;
  userId: string;
  accountId: string;
  userName: string;
  theme: string;
  isLoading: boolean;
}

const MondayContext = createContext<MondayContextType | null>(null);

// Detect local development (not inside monday.com iframe)
// Adapt the dev-mode check to your build tool:
//   Vite: import.meta.env.DEV
//   Webpack/CRA: process.env.NODE_ENV === "development"
//   Other: use your build tool's equivalent
const isLocalDev =
  typeof window !== "undefined" &&
  !window.location.ancestorOrigins?.length &&
  process.env.NODE_ENV === "development";

export function MondayProvider({ children }: { children: React.ReactNode }) {
  const [userId, setUserId] = useState("");
  const [accountId, setAccountId] = useState("");
  const [userName, setUserName] = useState("");
  const [theme, setTheme] = useState("light");
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    if (isLocalDev) {
      setUserId("test-user-1");
      setAccountId("test-account-1");
      setUserName("Local Developer");
      setTheme("light");
      setIsLoading(false);
      return;
    }

    monday.get("context").then((res: any) => {
      const ctx = res.data;
      setUserId(String(ctx.user?.id || ""));
      setAccountId(String(ctx.account?.id || ""));
      setUserName(ctx.user?.name || "");
      setTheme(ctx.theme || "light");
      setIsLoading(false);
    });

    monday.listen("context", (res: any) => {
      if (res.data?.theme) setTheme(res.data.theme);
    });
  }, []);

  return (
    <MondayContext.Provider
      value={{ monday, userId, accountId, userName, theme, isLoading }}
    >
      {children}
    </MondayContext.Provider>
  );
}

export function useMondayContext() {
  const ctx = useContext(MondayContext);
  if (!ctx) throw new Error("useMondayContext must be used within MondayProvider");
  return ctx;
}
```

**Wrap the existing app root** with `<MondayProvider>`:
- Find the main entry point (e.g., `main.tsx`, `index.tsx`, `App.tsx`)
- Wrap the top-level component with `<MondayProvider>`
- Do NOT restructure existing component hierarchy — just wrap it

#### 4.4: API Service Layer

**Create `src/services/api.ts`** (if the app has a backend):
```typescript
import mondaySdk from "monday-sdk-js";

const monday = mondaySdk();

// Adapt the dev-mode check to your build tool:
//   Vite: import.meta.env.DEV
//   Webpack/CRA: process.env.NODE_ENV === "development"
const isLocalDev =
  typeof window !== "undefined" &&
  !window.location.ancestorOrigins?.length &&
  process.env.NODE_ENV === "development";

async function getSessionToken(): Promise<string> {
  if (isLocalDev) {
    // Use your build tool's env var convention (e.g., VITE_DEV_TOKEN, REACT_APP_DEV_TOKEN)
    return "dev-token";
  }
  const token = await monday.get("sessionToken");
  return token.data;
}

async function getBackendUrl(): Promise<string> {
  if (isLocalDev) {
    return "http://localhost:8080";
  }
  const response = await monday.get("context");
  // @ts-ignore
  return response.data.appVersion.mondayCodeHostingUrl;
}

export async function apiFetch(path: string, options: RequestInit = {}) {
  const token = await getSessionToken();
  const backendUrl = await getBackendUrl();
  const res = await fetch(`${backendUrl}${path}`, {
    ...options,
    headers: {
      "Content-Type": "application/json",
      Authorization: token,
      ...options.headers,
    },
  });
  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}
```

Then guide the user on replacing their existing API calls (e.g., `fetch("/api/...")` or axios calls) with `apiFetch(...)`.

#### 4.5: monday-code CDN Entry Point

**Create `index.html`** in the frontend root (required by monday-code for CDN deployments):
```javascript
// This file is required by monday-code for CDN deployments
// It serves the built static files
```

#### 4.6: Update package.json Scripts

Ensure a `deploy` script exists. Keep the existing `dev` and `build` scripts as-is (they already work with the project's build tool). Add only the deploy script:

```json
{
  "scripts": {
    "deploy": "mapps code:push -c -d dist -a ${MONDAY_APP_ID:?} --force && rm -f dist.zip"
  }
}
```

**IMPORTANT**: The `build` script must output static files to a `dist/` directory. If the existing build outputs to a different directory (e.g., `build/`, `out/`, `public/`), either:
- Update the build config to output to `dist/`, OR
- Update the `deploy` script's `-d` flag to point to the actual output directory

#### 4.7: Add Vibe Design System (optional)

If the user wants monday.com native look and feel:
```bash
npm install @vibe/core
```

Suggest replacing existing UI component libraries (MUI, Chakra, etc.) with Vibe equivalents where it makes sense, but do NOT auto-replace — this is a separate task the user should opt into.

### Step 5: Backend Migration (if backend or fullstack)

#### 5.1: Directory Structure

If the backend is not in a `backend/` directory, ask the user whether to restructure.

#### 5.2: monday-code Entry Points

**Create `preload.cjs`** (for dotenv in local dev):
```javascript
require('dotenv').config();
```

**Create `index.js`** (monday-code serverless entry point):
```javascript
import path from "path";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));
const { default: app } = await import(path.join(__dirname, "dist", "app.js"));

const PORT = process.env.PORT || 8080;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**IMPORTANT**: The existing server start code (e.g., `app.listen(...)`) must be separated:
- `app.ts` should export the Express app WITHOUT calling `.listen()`
- `server.ts` should import the app and call `.listen()` (for local dev)
- `index.js` should import the compiled app and call `.listen()` (for monday-code)

If the existing code has `app.listen()` in the same file as route definitions, split it:
1. Extract route/middleware setup into `app.ts` (export default app)
2. Create `server.ts` that imports app and starts listening
3. Update `index.js` to import from `dist/app.js`

#### 5.3: Authentication Middleware
##### 5.3.1: For verifying monday session tokens
This is required for any backend that needs to authenticate requests from the frontend. If the existing app has auth middleware, it must be updated to support the JWT format used by monday.com session tokens

**Create `src/middleware/auth.ts`** for JWT verification of monday session tokens:
```typescript
import jwt from "jsonwebtoken";
import type { Request, Response, NextFunction } from "express";

export interface AuthContext {
  userId: string;
  accountId: string;
}

declare global {
  namespace Express {
    interface Request {
      auth?: AuthContext;
    }
  }
}

export function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith("Bearer ")) {
    return res.status(401).json({ error: "Missing authorization token" });
  }

  const token = authHeader.split(" ")[1];
  const secret = getSecret("MONDAY_CLIENT_SECRET");

  if (!secret) {
    console.error("MONDAY_CLIENT_SECRET not configured");
    return res.status(500).json({ error: "Server misconfigured" });
  }

  try {
    const decoded = jwt.verify(token, secret) as any;

    if (decoded.dat) {
      req.auth = {
        userId: String(decoded.dat.user_id),
        accountId: String(decoded.dat.account_id),
      };
    } else {
      req.auth = {
        userId: String(decoded.userId),
        accountId: String(decoded.accountId),
      };
    }

    next();
  } catch (err) {
    return res.status(401).json({ error: "Invalid token" });
  }
}
```


##### 5.3.2: For verifying automations/webhooks from monday.com
If the app needs to verify incoming requests from monday.com automations or webhooks, additional middleware is needed to verify the HMAC signature using the `MONDAY_CLIENT_SECRET`. This is separate from the session token verification and should be applied only to the specific routes that handle automation/webhook requests.

```typescript
import jwt from "jsonwebtoken";
import type { Request, Response, NextFunction } from "express";

/** Define the session property on the request object   */
declare global {
  namespace Express {
    interface Request {
      session: {
        accountId: string;
        userId: string;
        backToUrl: string | undefined;
        shortLivedToken: string | undefined;
      };
    }
  }
}

export default async function authenticationMiddleware(
  req: Request,
  res: Response,
  next: NextFunction
) {
  try {
    const authorization = req.headers.authorization ?? req.query?.token;

    if (typeof authorization !== "string") {
      res
        .status(401)
        .json({ error: "not authenticated, no credentials in request" });
      return;
    }

    if (typeof process.env.MONDAY_SIGNING_SECRET !== "string") {
      res.status(500).json({ error: "Missing MONDAY_SIGNING_SECRET (should be in .env file)" });
      return;
    }

    const { accountId, userId, backToUrl, shortLivedToken } = jwt.verify(
      authorization,
      getSecret("MONDAY_SIGNING_SECRET")
    ) as any;

    req.session = { accountId, userId, backToUrl, shortLivedToken };

    // shortLivedToken is only included for automations, not for regular session tokens. If it's present, you can use it to make authenticated api graphql requests back to monday.com on behalf of the user who triggered the automation.

    next();
  } catch (err) {
    res
      .status(401)
      .json({ error: "authentication error, could not verify credentials" });
  }
}
```


**If the app already has auth middleware:**
- Check if it handles monday JWT format (both `decoded.dat` and `decoded.userId` patterns)
- If not, update it to support both formats
- If it uses a different auth system (e.g., Firebase, Auth0), keep it alongside monday auth — the monday middleware handles the iframe session token, not user login

#### 5.4: Database Migration

**If using MongoDB/Mongoose:**
- Update connection string to use `MNDY_MONGODB_CONNECTION_STRING` env var
- This is auto-injected by monday-code after first deploy
- For local dev, fall back to a local MongoDB instance
- Add `accountId` filtering to all queries for multi-tenant isolation

**If using PostgreSQL/MySQL/SQLite:**
- monday-code only provides MongoDB (Document DB) natively so you will have to migrate to MongoDB to use monday-code's managed database, OR keep using your existing database externally and connect to it from the monday-code backend
- If keeping external DB, set connection string via `mapps code:env` or secrets. FYI this option is discouraged due to increased latency and security considerations of connecting to an external DB from a mondday.com environment

**Multi-tenant isolation** — ALL database queries must filter by `accountId`:
```typescript
interface BaseDocument {
  accountId: string;
  ownerId: string;
  createdAt: string;
  updatedAt: string;
}

// CORRECT
const items = await collection.find({ accountId: req.auth!.accountId }).toArray();

// WRONG — exposes all tenants' data
const items = await collection.find({}).toArray();
```

Scan existing queries and flag any that don't filter by tenant.

#### 5.5: monday-code SDK Utilities

**Add `@mondaycom/apps-sdk`:**
```bash
npm install @mondaycom/apps-sdk
```

`src/utils/secrets.ts`:

Fetching secrets and environment variables should use the monday-code SDK utilities to ensure compatibility with monday-code's secret management and environment variable injection.

```typescript
import { SecretsManager } from "@mondaycom/apps-sdk";

const secretsManager = new SecretsManager();

export async function getSecret(key: string): Promise<string | undefined> {
  try {
    const { value } = await secretsManager.getSecret(key);
    return value;
  } catch {
    return process.env[key];
  }
}
```

Then replace all `process.env.KEY` usages in the backend code with `getSecret("KEY")` to ensure it works both in local dev and when deployed to monday-code.

`src/utils/env-vars.ts`:
```typescript
import { EnvironmentVariablesManager } from "@mondaycom/apps-sdk";

const envManager = new EnvironmentVariablesManager();

export function getEnvVar(key: string): string {
  const value = envManager.getEnvVar(key);
  if (!value) {
    throw new Error(`Environment variable not set: ${key}`);
  }
  return value;
}
```

`src/utils/logger.ts`:

Logging should also use the monday-code Logger for better integration with monday-code's logging system.

```typescript
import { Logger } from "@mondaycom/apps-sdk";

export function createLogger(tag: string) {
  return new Logger(tag);
}
```

Then replace any `console.log` or other logging calls with the created logger instance (e.g., `logger.info(...)`, `logger.error(...)`).

#### 5.6: Update package.json

Ensure these scripts and dependencies exist:
```json
{
  "type": "module",
  "scripts": {
    "build": "rm -rf dist && tsc",
    "dev": "tsx watch --require ./preload.cjs src/server.ts",
    "start": "node --require ./preload.cjs index.js",
    "deploy": "mapps code:push -a ${MONDAY_APP_ID:?} --force && rm -f code.tar.gz"
  }
}
```

Add missing dependencies:
```bash
npm install @mondaycom/apps-sdk jsonwebtoken dotenv
npm install -D @types/jsonwebtoken tsx
```

### Step 6: Environment Files

**Create `.env.example`** in relevant directories:

Backend:
```bash
MNDY_MONGODB_CONNECTION_STRING=mongodb://localhost:27017/{{APP_NAME}}
MONDAY_CLIENT_SECRET=your-client-secret
PORT=8080
```

Frontend (use the env var prefix convention for your build tool — `VITE_` for Vite, `REACT_APP_` for CRA, etc.):
```bash
DEV_TOKEN=your-dev-jwt-token
```

Copy to `.env` if no `.env` exists. If `.env` already exists, merge new variables into it without overwriting existing values.

### Step 7: Migration Checklist

After completing the migration, present a checklist to the user:

**Root:**
- [ ] `.mondaycoderc` exists with valid runtime
- [ ] `manifest.json` exists with correct feature type

**Frontend (if applicable):**
- [ ] Build tool configured with `global = globalThis` define
- [ ] Build outputs static files to `dist/` (or deploy script adjusted)
- [ ] `monday-sdk-js` installed
- [ ] `MondayContext` provider wraps the app
- [ ] `index.js` CDN entry point exists
- [ ] Build succeeds (`npm run build`)
- [ ] `api.ts` service uses monday session tokens (if fullstack)

**Backend (if applicable):**
- [ ] `"type": "module"` in package.json
- [ ] `app.ts` exports app WITHOUT calling `.listen()`
- [ ] `server.ts` handles local dev startup
- [ ] `index.js` serverless entry point exists
- [ ] `preload.cjs` exists for dotenv
- [ ] Auth middleware handles monday JWT format
- [ ] `@mondaycom/apps-sdk` installed
- [ ] Database queries filter by `accountId`
- [ ] Build succeeds (`npm run build`)

**Next steps:**
- [ ] Create app at `https://<slug>.monday.com/developers/apps`
- [ ] Set `MONDAY_CLIENT_SECRET` in `.env`
- [ ] Run `/monday-code-deploy:monday-deploy` to deploy

### Step 8: Build Verification

After migration, run builds to verify everything compiles:

```bash
# Frontend
cd frontend && npm run build

# Backend
cd backend && npm run build
```

If the project is using TypeScript, fix any TypeScript or build errors before considering migration complete.

### Adding Multi-Tenant Isolation to Existing Queries
- Search for all database queries (`.find(`, `.findOne(`, `.aggregate(`, `.updateOne(`, etc.)
- Ensure each query includes `accountId: req.auth!.accountId` in the filter
- Add `accountId` and `ownerId` fields to all document insert operations
- Create database indexes on `accountId` for performance

## Troubleshooting

**Build fails after migration: `global is not defined`**
— The build tool is missing the `global = globalThis` define. See Step 4.2 for build-tool-specific fixes.

**`app.listen is not a function` or server won't start on monday-code**
— The `app.listen()` call must be separated from route definitions. `app.ts` must export the app without calling `.listen()`. See Step 5.2.

**`MONDAY_CLIENT_SECRET` is undefined at runtime**
— Use `getSecret("MONDAY_CLIENT_SECRET")` from `apps-sdk` instead of `process.env`. Make sure the secret is set via `mapps code:env:set`.

**MongoDB connection fails after deploy**
— Use `MNDY_MONGODB_CONNECTION_STRING` env var (auto-injected after first deploy). Do not hardcode the connection string.

**TypeScript compilation errors after restructuring**
— Check that `tsconfig.json` includes the new files and that all imports use the updated file paths. Run `tsc --noEmit` to see all errors before deploying.

**Session token returns 401 in production**
— The auth middleware must handle both `decoded.dat.user_id` (monday JWT format) and `decoded.userId` (legacy). See Step 5.3.

## Notes

- NEVER delete or overwrite existing business logic — only add monday-code compatibility
- Always present the migration plan to the user before making changes
- If the existing project has tests, verify they still pass after migration
- If the project uses a CI/CD pipeline, note that deployment scripts may need updating
- monday-code CDN only serves static files — any SSR must move to the backend
- monday-code serverless has cold start times — optimize imports for fast startup
