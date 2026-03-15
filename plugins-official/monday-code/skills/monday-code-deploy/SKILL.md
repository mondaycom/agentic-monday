---
name: monday-code-deploy
description: Build, deploy, and manage monday code apps with multi-region, cron, alerts, and security scanning. Use when user says "deploy my app", "push to monday-code", "deploy to monday", "check deployment status", "set environment variables", "push my app", "deploy backend", "deploy frontend", or wants to promote an app version.
argument-hint: "[frontend|backend|all|status|env|cron|alerts]"
user-invocable: true
allowed-tools: ["Bash", "Read", "Write", "Glob", "Grep", "mcp__monday-apps__*"]
---

# monday-deploy

Build and deploy monday code apps to the monday-code platform (serverless + CDN).

## When to Use

- User wants to deploy to monday-code
- User wants to check deployment status
- User wants to manage environment variables or secrets
- User wants to set up cron jobs
- User wants to configure alerts
- User wants to run security scans
- User wants to promote an app version

## Instructions
### Step 0: Prerequisites
- Your monday account must have access to monday code. This is available OOTB on developer tier accounts, but must be opted into for other tiers. If you don't see "Code" in the left sidebar of the Developer Center, contact your monday admin to enable it. Free accounts do not have access to monday code.
- You must have the `mapps` (`npm -g i @mondaycom/apps-cli`) CLI installed and authenticated (`mapps init`). This is the command-line tool for interacting with the monday-code platform.
- You will need to go to the Developer Center and create a new app 
- Don't forget to set up oauth scopes in the Developer Center if your app requires them, before promoting the app to live and publishing to customers.
- An initial deployment is required to set up the environment variables (secrets) and get the auto-injected `MNDY_MONGODB_CONNECTION_STRING` for database access. Deploying an empty app first is recommended to get this value, which is needed for backend deployments.

### Step 1: Pre-Deployment Checks
You will need to have the monday app id available as an environment variable (`MONDAY_APP_ID`) to deploy on your local machine. This is the unique identifier for your app in the monday ecosystem. The app ID will be displayed on the "General Settings" page of your app in the Developer Center.

**Verify MONDAY_APP_ID:**
```bash
echo "MONDAY_APP_ID=${MONDAY_APP_ID}"
```
If not set, ask the user. They can find it in the URL of their app in the Developer Center.

**Verify mapps CLI is installed and authenticated:**
```bash
mapps --version
```

If not authenticated:
```bash
mapps init
```

**Build and verify:**
```bash
# Frontend
cd frontend && npm run build

# Backend
cd backend && npm run build
```

Fix any TypeScript errors before deploying.

### Step 2: Deploy

**Frontend deployment (CDN):**
```bash
cd frontend
npm run build
mapps code:push -c -d dist -a ${MONDAY_APP_ID:?} --force && rm -f dist.zip
```

Flags:
- `-c` = CDN deployment (client-side static files)
- `-d dist` = Use the dist directory
- `--force` = Override existing deployment (required to push directly to the live version, without the need to create a draft version first, pushing and promoting)

**Backend deployment (Serverless):**
```bash
cd backend
npm run build
mapps code:push -a ${MONDAY_APP_ID:?} --force && rm -f code.tar.gz
```

**Fullstack deployment (both):**
Deploy frontend first, then backend:
```bash
# 1. Frontend to CDN
cd frontend && npm run build && mapps code:push -c -d dist -a ${MONDAY_APP_ID:?} --force && rm -f dist.zip

# 2. Backend to serverless
cd ../backend && npm run build && mapps code:push -a ${MONDAY_APP_ID:?} --force && rm -f code.tar.gz
```

**With security scanning:**
Add `-s` flag to enable security scanning of the deployment artifact:
```bash
mapps code:push -a ${MONDAY_APP_ID:?} -s
```

View the security report after deployment:
```bash
mapps code:report -a ${MONDAY_APP_ID:?}
```

Security scanning is non-blocking - deployment proceeds even if vulnerabilities are found.

### Step 3: Connect Deployment to App Features

After deployment, connect it to app features

```bash
mapps app-features:build -a ${MONDAY_APP_ID} -i <version_id> -f <feature_id> -d
```

Feature types and deployment mapping:
| Feature Type | Deployment | Flag |
|---|---|---|
| BoardView, ItemView, DashboardWidget, ProductView | CDN | `-c` |
| Workflow block | Serverless | (no flag) |

### Step 4: Verify Deployment

Check deployment status:
```bash
mapps code:status --appVersionId <version_id>
```

Or use MCP:
```
monday_apps_get_deployment_status({ appVersionId: VERSION_ID })
```

View logs:
```bash
# Get version ID first
mapps code:logs -i <version_id>

# Stream real-time logs
mapps code:logs -i <version_id> --follow
```

### Step 5: Environment Variables and Secrets

**List current env vars:**
```bash
mapps code:env -a ${MONDAY_APP_ID}
```

**Set environment variable:**
```bash
mapps code:env -a ${MONDAY_APP_ID} -k KEY -v "value"
```

**Common secrets (as environmnet variables) to set in production:**
- `MONDAY_CLIENT_SECRET` - Required for JWT auth for a fullstack app(set in Developer Center > OAuth)
- `MONDAY_SIGNING_SECRET` - Required for verifying webhooks from automations


**Note:** `MNDY_MONGODB_CONNECTION_STRING` is auto-injected by monday-code after first deploy. Do NOT set it manually.

**Using Secrets Manager** (for sensitive values accessed at runtime):
```typescript
import { SecretsManager } from "@mondaycom/apps-sdk";
const secrets = new SecretsManager();
const { value } = await secrets.getSecret("MONDAY_CLIENT_SECRET");
```

** Set a secret value:**
Either via the UI in the Developer Center > App > Host on monday > Server-side code > Secrets tab,

Or via CLI:
```bash
mapps secrets:set -a ${MONDAY_APP_ID} -k MONDAY_CLIENT_SECRET -v "your_client_secret_value"
```

## Advanced Features

For advanced features (multi-region deployment, cron jobs, alerts, promoting app versions), read `references/advanced-features.md`.

Use advanced features when:
- User asks about deploying to multiple regions (us, eu, au, il)
- User wants to schedule recurring background jobs
- User wants to monitor their app with automated alerts
- User wants to promote a development version to live

## Outbound Networking

If your app calls external APIs, you may need to configure an allowlist:

- monday-code provides static IPs per region for outbound traffic
- Allowlist uses CIDR/FQDN format (no wildcards)
- Changes apply immediately once activated

Configure in Developer Center > App > Outbound Communication.

## Deployment Workflow Summary

1. Build locally and verify no errors
2. Run tests: `/monday-test`
3. Deploy: `mapps code:push`
4. Connect to features: `mapps app-features:build`
5. Set env vars: `mapps code:env`
6. Verify: check status + logs
7. (Optional) Security scan: `mapps code:push -s`
8. (Optional) Advanced features: see `references/advanced-features.md`
9. Promote to live when ready

## Usage Example

**User says:** "Deploy my monday app to production"

**Actions:**
1. Check `MONDAY_APP_ID` is set and `mapps` CLI is authenticated
2. Build frontend: `cd frontend && npm run build`
3. Push frontend to CDN: `mapps code:push -c -d dist -a ${MONDAY_APP_ID} --force`
4. Build backend: `cd backend && npm run build`
5. Push backend to serverless: `mapps code:push -a ${MONDAY_APP_ID} --force`
6. Verify deployment: `mapps code:status --appVersionId <version_id>`

**Result:** App is live on monday-code with frontend on CDN and backend on serverless infrastructure.

## Troubleshooting

**mapps not authenticated / `mapps init` required:**
- Run `mapps init` and follow the prompts to authenticate with your monday account.

**MONDAY_APP_ID not set:**
- Find the app ID in Developer Center > General Settings > App ID, then run `export MONDAY_APP_ID=<id>`.

**Build fails (TypeScript errors):**
- Fix all TypeScript errors before deploying. Run `npm run build` locally and resolve any compilation errors.

**Deployment times out:**
- Large bundles can time out. Check that `node_modules/` is excluded from the deployment artifact. The backend uses code.tar.gz which should only include built files.

**`code:push` fails with "app not found":**
- Verify the app ID matches the app in Developer Center. Ensure you are authenticated with the correct monday account that owns the app.

**Environment variable not visible after deploy:**
- Re-deploy after setting env vars with `mapps code:env`. Environment variables are baked in at deploy time.

**`MNDY_MONGODB_CONNECTION_STRING` is undefined:**
- This is auto-injected after the first deploy. Deploy an empty app first, then add your code and deploy again.

## Notes

- Always build before deploying
- Use MCP tools (`mcp__monday-apps__*`) to get app version and feature IDs
- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected - never set manually
- Security scanning is non-blocking (deploy proceeds regardless)
- Clean up build artifacts (dist.zip, code.tar.gz) after deploy
- See `references/advanced-features.md` for multi-region, cron jobs, alerts, and promoting versions
