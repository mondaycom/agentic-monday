---
name: setup
description: Bootstrap a project using the monday.com public GraphQL API. Installs the @mondaydotcomorg/api SDK, configures auth, pins API version, and sets up TypeScript codegen. Use when starting a new API integration or adding monday.com API to an existing project.
argument-hint: "<project-path> [typescript|javascript]"
user-invocable: true
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion]
---

# API Setup

Bootstrap a project to use the monday.com public GraphQL API with the `@mondaydotcomorg/api` SDK.

## Scope

- Helps external developers set up a new or existing Node.js project for the monday.com API
- Does NOT set up monday-code apps or internal platform API — those are separate plugins

## Prerequisites

- Node.js 18+ installed
- A monday.com API token

**To get a personal API token:**
1. In monday.com, click your profile picture → **Developers** → **API**
2. Click **Show** next to your Personal API token
3. Copy the token — treat it like a password

**App tokens** (for production apps installed by other users) require creating an app in the Developer Center and implementing OAuth. For initial development, a personal token is fine.

---

## Workflow

### Step 1: Detect Project State

Check the current project:
- Does `package.json` exist? If not, ask if they want to initialize one (`npm init -y`)
- Is `@mondaydotcomorg/api` already installed? If yes, check version and suggest upgrade if < v14
- Is TypeScript configured (`tsconfig.json`)? This affects which packages to install
- Which package manager? Detect from lockfiles:
  - `yarn.lock` → use `yarn add`
  - `pnpm-lock.yaml` → use `pnpm add`
  - `package-lock.json` → use `npm install`
  - None found → default to npm

### Step 2: Install Dependencies

**JavaScript projects:**

```bash
npm install @mondaydotcomorg/api
```

**TypeScript projects:**

```bash
npm install @mondaydotcomorg/api
npm install -D @mondaydotcomorg/api-types
```

For full codegen setup (typed query results):

```bash
npx @mondaydotcomorg/setup-api
```

This scaffolds:
- `codegen.yml` — graphql-codegen config
- `graphql.config.yml` — IDE extension config
- `src/queries.graphql.ts` — starter query file
- `fetch-schema.sh` — script to pull the live schema
- Scripts in `package.json`: `fetch:schema`, `codegen`, `fetch:generate`

### Step 3: Configure Authentication

Create a `.env` file (if it doesn't exist) and add:

```
MONDAY_API_TOKEN=your_token_here
```

Add `.env` to `.gitignore` if not already listed.

**Node.js does not load `.env` files automatically.** Install `dotenv` and load it at the entry point of the app:

```bash
npm install dotenv
```

```typescript
import 'dotenv/config'; // must be first import
import { ApiClient } from '@mondaydotcomorg/api';
```

Alternatively, with Node 20.6+ you can pass `--env-file=.env` to the node CLI without any package.

**Key points to explain:**
- The token mirrors the **exact permissions** of the user who created it. If the user can't see a board in the UI, API calls can't see it either.
- Do NOT prefix with `Bearer` — the SDK handles this. For raw HTTP, pass the token directly as the `Authorization` header value.
- **Token regeneration is destructive** — regenerating a token instantly invalidates the old one. All integrations using it break.

### Step 4: Create SDK Initialization Code

**Server-side (Node.js):**

```typescript
import { ApiClient } from '@mondaydotcomorg/api';

const client = new ApiClient({
  token: process.env.MONDAY_API_TOKEN!,
  apiVersion: '2026-01'
});

// Usage
const result = await client.request(`{ me { name email } }`);
console.log(result.me.name);
```

**Raw HTTP (no SDK):**

```typescript
const response = await fetch('https://api.monday.com/v2', {
  method: 'POST',
  headers: {
    'Authorization': process.env.MONDAY_API_TOKEN!,
    'Content-Type': 'application/json',
    'API-Version': '2026-01'
  },
  body: JSON.stringify({
    query: '{ me { name email } }'
  })
});

const { data, errors } = await response.json();
```

### Step 5: Pin API Version

Always pin to the current stable version: **`2026-01`**

Explain:
- Versions rotate quarterly: `01`, `04`, `07`, `10`
- If you omit the version header, you get the Current version — but when it rotates, your app silently gets the new version which may have breaking changes
- **Always pin explicitly** — never rely on the default
- Lifecycle: Release Candidate (testing) → Current (stable) → Maintenance (old but works) → Deprecated (may stop working)

### Step 6: Verify Setup

Provide a test query and offer to run it live:

```typescript
const result = await client.request(`{ me { name email account { name } } }`);
console.log('Connected as:', result.me.name);
console.log('Account:', result.me.account.name);
```

Offer to verify via `mcp__monday__all_monday_api`:

```graphql
{ me { name email account { name } } }
```

If this succeeds, the setup is complete.

### Step 7: Suggest Next Steps

- Use `/monday-api:build-query` to start writing queries
- Use `/monday-api:ops` for production readiness guidance
- Review the SDK docs: `@mondaydotcomorg/api` on npm

---

## Key Gotchas

| Gotcha | Details |
|--------|---------|
| Node version | 18+ required for `@mondaydotcomorg/api` v14 (uses native `fetch`) |
| No Bearer prefix | SDK adds auth automatically; raw HTTP uses `Authorization: <token>` directly |
| Token permissions | Token has exactly the same access as the user who created it — no elevated API access |
| API version | Must be pinned; defaults change quarterly and can break your app silently |
| pnpm support | `npx @mondaydotcomorg/setup-api` may not detect pnpm — install deps manually if needed |
| SeamlessApiClient | Not for external integrations — only for apps embedded inside the monday.com UI (iframe). Requires monday.com parent frame context. |
| App vs personal token | Personal tokens are fine for dev. For production apps used by others, use app tokens (OAuth). Personal tokens expose your own account's data only. |

## Error Handling

| Error | Action |
|-------|--------|
| `npm install` fails | Check Node.js version (18+), check network access to npm registry |
| 401 on test query | Token is invalid or expired — regenerate in monday.com Developer section |
| "Cannot find module" | Check import path — use `@mondaydotcomorg/api` not `monday-sdk-js` (legacy) |
| codegen fails | Run `yarn fetch:schema` first, ensure Node 18+ (SDK) or Node 20+ (codegen type generation) |
