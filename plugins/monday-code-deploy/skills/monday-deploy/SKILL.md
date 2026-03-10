---
name: monday-deploy
description: Build, deploy, and manage monday code apps with multi-region, cron, alerts, and security scanning
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

### Step 1: Pre-Deployment Checks

**Verify MONDAY_APP_ID:**
```bash
echo "MONDAY_APP_ID=${MONDAY_APP_ID}"
```
If not set, ask the user. They can find it in the URL of their app in the Developer Center.

**Verify mapps CLI is installed and authenticated:**
```bash
npx @mondaycom/apps-cli --version
```

If not authenticated:
```bash
npx @mondaycom/apps-cli init
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
- `--force` = Override existing deployment

**Backend deployment (Serverless):**
```bash
cd backend
npm run build
mapps code:push -a ${MONDAY_APP_ID:?} --force && rm -f code.tar.gz
```

No `-c` flag = serverless deployment.

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
mapps code:push -a ${MONDAY_APP_ID:?} --force -s
```

View the security report after deployment:
```bash
mapps code:report -a ${MONDAY_APP_ID:?}
```

Security scanning is non-blocking - deployment proceeds even if vulnerabilities are found.

### Step 3: Connect Deployment to App Features

After deployment, connect it to app features. Use MCP tools if available:

```
monday_apps_get_app_features({ appId: MONDAY_APP_ID, appVersionId: VERSION_ID })
```

Or via CLI:
```bash
mapps app-features:build -a ${MONDAY_APP_ID} -i <version_id> -f <feature_id> -d
```

Feature types and deployment mapping:
| Feature Type | Deployment | Flag |
|---|---|---|
| BoardView, ItemView, DashboardWidget, ProductView | CDN | `-c` |
| Integration, Webhook, AI feature | Serverless | (no flag) |

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

**Common env vars to set in production:**
- `MONDAY_CLIENT_SECRET` - Required for JWT auth (set in Developer Center > OAuth)
- `NODE_ENV` - Set to `"production"`

**Note:** `MNDY_MONGODB_CONNECTION_STRING` is auto-injected by monday-code after first deploy. Do NOT set it manually.

**Using Secrets Manager** (for sensitive values accessed at runtime):
```typescript
import { SecretsManager } from "@mondaycom/apps-sdk";
const secrets = new SecretsManager();
const { value } = await secrets.getSecret("API_KEY");
```

Or use MCP:
```
monday_apps_set_environment_variable({ appId: APP_ID, key: "KEY", value: "val" })
monday_apps_list_environment_variable_keys({ appId: APP_ID })
```

## Advanced: Multi-Region Deployment

monday-code supports 4 regions: **US, EU, AU, IL**.

**Important constraints:**
- Multi-region must be enabled BEFORE using Document DB (irreversible)
- Public apps only (not private/dev apps)
- Environment variables are set per-region
- Alerts are global (apply to all regions)
- Cron jobs are per-region (IL not supported for cron)

Enable multi-region in the Developer Center before first production deploy.

**Setting env vars per region:**
```bash
mapps code:env -a ${MONDAY_APP_ID} -k KEY -v "value" -r US
mapps code:env -a ${MONDAY_APP_ID} -k KEY -v "value" -r EU
```

## Advanced: Cron Jobs

Schedule recurring jobs (max 5 per region, IL not supported):

**Create a cron route in your backend:**
```typescript
// POST /mndy-cronjob/daily-cleanup
app.post("/mndy-cronjob/daily-cleanup", async (req, res) => {
  // Cron job logic here
  await cleanupOldRecords();
  res.json({ success: true });
});
```

**IMPORTANT**: Cron routes MUST use the path prefix `/mndy-cronjob/`.

**Schedule the job:**
```bash
mapps scheduler:create -a ${MONDAY_APP_ID} -s "0 0 * * *" -u "/mndy-cronjob/daily-cleanup" -r US
```

Cron expression format: `minute hour day-of-month month day-of-week`

**List scheduled jobs:**
```bash
mapps scheduler:list -a ${MONDAY_APP_ID}
```

**Delete a job:**
```bash
mapps scheduler:delete -a ${MONDAY_APP_ID} -j <job_id>
```

## Advanced: Alerts

Monitor your app with automated alerts (creates a monday.com board):

**3 alert types:**
1. **HTTP error rate** - Triggers when error rate exceeds threshold
2. **HTTP latency** - Triggers when response time exceeds threshold
3. **Runtime limit quota** - Triggers when approaching compute limits

**Create alerts via Developer Center** > App > Alerts section.

Alerts are global (apply to all regions). They auto-create a monday.com board for notifications.

## Advanced: Promoting App Versions

Promote a development version to live:

```
monday_apps_promote_app({ appId: APP_ID, appVersionId: VERSION_ID })
```

Or through the Developer Center UI.

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
8. (Optional) Set up cron jobs
9. (Optional) Configure alerts
10. Promote to live when ready

## Notes

- Always build before deploying
- Use MCP tools (`mcp__monday-apps__*`) to get app version and feature IDs
- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected - never set manually
- Security scanning is non-blocking (deploy proceeds regardless)
- Cron routes must be POST handlers at `/mndy-cronjob/<path>`
- Enable multi-region BEFORE first Document DB usage
- Clean up build artifacts (dist.zip, code.tar.gz) after deploy
