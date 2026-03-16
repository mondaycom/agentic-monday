---
name: monday-code-ops
description: "Debug, monitor, and operate monday code apps in production. Use when user wants to 'check logs', 'get deployment URL', 'view environment variables', 'get connection string', 'check alerts', 'debug production', 'get app status', 'troubleshoot monday code', or needs help with any production operational task."
argument-hint: "[logs|status|env|secrets|db|alerts|urls]"
user-invocable: true
---

# monday-code-ops

Debug, monitor, and operate monday code apps after deployment. This skill covers everything needed to work with a live monday code app: viewing logs, getting deployment URLs, managing environment variables and secrets, retrieving database connection strings, monitoring alerts, and troubleshooting production issues.

## When to Use

- User wants to view or stream production logs
- User wants to get the deployment URLs (CDN or serverless hosting URL)
- User wants to check deployment status
- User wants to manage environment variables or secrets in production
- User wants to get the MongoDB connection string for direct database access
- User wants to check or configure alerts on the alerts board
- User wants to debug a production issue
- User wants to check cron job status or run a job on demand
- User wants to get a security scan report

## Usage Examples

```
/monday-code-ops logs
/monday-code-ops status
/monday-code-ops env
/monday-code-ops secrets
/monday-code-ops db
/monday-code-ops alerts
/monday-code-ops urls
```

Or conversationally: "Show me the production logs", "What's the deployment URL?", "Get my MongoDB connection string", "Check the alerts board", "Why is my app failing in production?".

## Prerequisites

- `mapps` CLI installed (`npm -g i @mondaycom/apps-cli`) and authenticated (`mapps init`)
- `MONDAY_APP_ID` environment variable set (find it in Developer Center > General Settings)
- App must be deployed at least once to monday-code

Verify prerequisites:
```bash
mapps --version
echo "MONDAY_APP_ID=${MONDAY_APP_ID}"
```

If `MONDAY_APP_ID` is not set, ask the user. They can find it in the Developer Center URL or General Settings page.

## Instructions

### 1. Get Deployment Status

Check the current deployment status and version info.

**`mapps code:status`** — Status of a specific project hosted on monday-code.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appVersionId` | The app version ID |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

```bash
# List all app versions to find the current version ID
mapps app-version:list -i ${MONDAY_APP_ID}

# Check deployment status for a specific version
mapps code:status -i <version_id>

# Check status for a specific region
mapps code:status -i <version_id> -z us
```

**`mapps app-version:list`** — List all versions for a specific app.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appId` | The app ID |

**`mapps app-version:builds`** — List all builds for a specific app version.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appVersionId` | The app version ID |

**`mapps app-features:list`** — List all features for a specific app version.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-i` | `--appVersionId` | The app version ID |

```bash
mapps app-features:list -a ${MONDAY_APP_ID} -i <version_id>
```

Or use MCP:
```
monday_apps_get_deployment_status({ appVersionId: VERSION_ID })
monday_apps_get_app_versions({ appId: APP_ID })
monday_apps_get_app_version({ appId: APP_ID, versionId: VERSION_ID })
monday_apps_get_app_features({ appId: APP_ID })
```

### 2. Get Deployment URLs

The deployment URLs are how the app is accessed in production:

**Frontend (CDN) URL:**
The CDN URL is not directly retrievable via CLI. It is available in:
- The Developer Center > App > Host on monday > Client-side code section

**Backend (Serverless) URL:**
To fetch the serverless hosting URL through the CLI:
```bash
# Get the serverless hosting URL (requires version ID)
mapps code:status -i <version_id>
```

- The monday SDK context: `monday.get("context")` returns `appVersion.mondayCodeHostingUrl` for the serverless URL
- Use MCP to get feature builds which contain the URLs:

```
monday_apps_get_app_features({ appId: APP_ID })
monday_apps_get_app_version({ appId: APP_ID, versionId: VERSION_ID })
```

The frontend accesses the backend URL through the monday SDK:
```typescript
const response = await monday.get("context");
const backendUrl = response.data.appVersion.mondayCodeHostingUrl;
```

### 3. View Logs

**`mapps code:logs`** — Stream logs from a monday code deployment.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appVersionId` | The app version ID |
| `-t` | `--logsType` | Log type: `http` for HTTP events, `console` for stdout |
| `-s` | `--eventSource` | Source: `live` for live events, `History` for past events |
| `-f` | `--logsStartDate` | Start date `MM/DD/YYYY HH:mm` (only with `-s live`) |
| `-e` | `--logsEndDate` | End date `MM/DD/YYYY HH:mm` (only with `-s live`) |
| `-r` | `--logSearchFromText` | Regex text search filter (only with `-s live`) |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

Global flags: `--verbose` (advanced logs), `--print-command` (print executed command).

```bash
# Stream live logs (default)
mapps code:logs -i <version_id>

# View only HTTP request logs
mapps code:logs -i <version_id> -t http

# View only console/stdout logs
mapps code:logs -i <version_id> -t console

# View live logs within a date range
mapps code:logs -i <version_id> -s live -f "03/15/2026 00:00" -e "03/16/2026 23:59"

# Search logs for a specific text pattern (regex)
mapps code:logs -i <version_id> -s live -r "error|timeout"

# View historical logs
mapps code:logs -i <version_id> -s History

# View logs for a specific region
mapps code:logs -i <version_id> -z eu

# Combine: search console logs in EU region within a date range
mapps code:logs -i <version_id> -t console -s live -f "03/15/2026 09:00" -e "03/15/2026 17:00" -r "uncaught" -z eu
```

**Getting the version ID:**
```bash
mapps app-version:list -i ${MONDAY_APP_ID}
```

Or use MCP:
```
monday_apps_get_app_versions({ appId: APP_ID })
```

### 4. Environment Variables

**`mapps code:env`** — Manage environment variables for an app hosted on monday-code.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appId` | The app ID |
| `-m` | `--mode` | Mode: `list-keys`, `set`, or `delete` |
| `-k` | `--key` | Variable key (required for `set` and `delete`) |
| `-v` | `--value` | Variable value (required for `set`) |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

```bash
# List all environment variable keys
mapps code:env -i ${MONDAY_APP_ID} -m list-keys

# Set an environment variable
mapps code:env -i ${MONDAY_APP_ID} -m set -k KEY_NAME -v "value"

# Delete an environment variable
mapps code:env -i ${MONDAY_APP_ID} -m delete -k KEY_NAME

# Set per region (multi-region apps)
mapps code:env -i ${MONDAY_APP_ID} -m set -k KEY_NAME -v "value" -z us
```

Or use MCP:
```
monday_apps_list_environment_variable_keys({ appId: APP_ID })
monday_apps_set_environment_variable({ appId: APP_ID, key: "KEY", value: "value" })
```

**Important:** Environment variables require a re-deploy to take effect. After setting env vars, re-deploy the app.

### 5. Secrets Management

**`mapps code:secret`** — Manage secret variables for an app hosted on monday-code.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appId` | The app ID |
| `-m` | `--mode` | Mode: `list-keys`, `set`, or `delete` |
| `-k` | `--key` | Variable key (required for `set` and `delete`) |
| `-v` | `--value` | Variable value (required for `set`) |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

```bash
# List all secret keys
mapps code:secret -i ${MONDAY_APP_ID} -m list-keys

# Set a secret
mapps code:secret -i ${MONDAY_APP_ID} -m set -k SECRET_KEY -v "secret_value"

# Delete a secret
mapps code:secret -i ${MONDAY_APP_ID} -m delete -k SECRET_KEY

# Set per region (multi-region apps)
mapps code:secret -i ${MONDAY_APP_ID} -m set -k SECRET_KEY -v "secret_value" -z us
```

**Common secrets:**
- `MONDAY_CLIENT_SECRET` - JWT auth for fullstack apps (from Developer Center > OAuth)
- `MONDAY_SIGNING_SECRET` - Webhook verification for automations

**Accessing secrets at runtime:**
```typescript
import { SecretsManager } from "@mondaycom/apps-sdk";
const secrets = new SecretsManager();
const { value } = await secrets.getSecret("MONDAY_CLIENT_SECRET");
```

### 6. Database Connection String

**`mapps database:connection-string`** — Get the connection string for your app database.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

```bash
# Get the connection string (for MongoDB Compass, mongosh, etc.)
mapps database:connection-string -a ${MONDAY_APP_ID}

# Get connection string for a specific region
mapps database:connection-string -a ${MONDAY_APP_ID} -z us
```

**Notes:**
- It takes about a 1-2 minutes after running the command for the connection string to actually become active and usable
- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected by monday-code after first deploy
- Max storage: 1 GiB per app
- Limits: 50,000 reads / 20,000 writes per region per day
- The connection string can be used with MongoDB Compass, mongosh, or other MongoDB tools for direct database inspection

### 7. Alerts and Monitoring

monday code alerts auto-create a monday.com board for notifications. See [references/alerts-and-monitoring.md](references/alerts-and-monitoring.md) for detailed setup.

**3 alert types:**
1. **HTTP error rate** - Triggers when error rate (4xx/5xx) exceeds threshold percentage within a time window
2. **HTTP latency response** - Triggers when response time exceeds threshold at a given percentile
3. **Runtime limit quota** - Triggers when approaching daily compute limits

**Setup alerts:**
1. Open your app in Developer Center
2. Navigate to Host on monday > Server-side code > Alert policies tab
3. Click "Create alert"
4. Select workspace for the alert board (first time only)
5. Configure alert type, threshold, and time window

**Working with the alerts board:**
- Alerts auto-populate a monday.com board with columns: Alert Name, Type, Status, Region, Timestamp, Details
- Use the monday MCP tools to query the alerts board:

```
mcp__monday__get_board_items_page({ boardId: ALERT_BOARD_ID })
mcp__monday__get_board_info({ boardId: ALERT_BOARD_ID })
```

- Build automations on the alert board to send Slack/email notifications, assign ownership, or trigger incident workflows
- Alert board ID can be found in the Developer Center > Alerts section

### 8. Cron Jobs Management

View and manage scheduled background jobs.

**`mapps scheduler:list`** — List all scheduled jobs.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

**`mapps scheduler:create`** — Create a new scheduled job.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-n` | `--name` | Job name (no whitespace) |
| `-s` | `--schedule` | Cron expression (relative to UTC) |
| `-e` | `--targetUrl` | Target URL path (relative to `/mndy-cronjob/<YOUR_ENDPOINT>`) |
| `-d` | `--description` | Job description (optional) |
| `-r` | `--maxRetries` | Maximum retries for failed jobs (optional) |
| `-b` | `--minBackoffDuration` | Min backoff duration in seconds between retries (optional) |
| `-t` | `--timeout` | Job execution timeout in seconds (optional) |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

**`mapps scheduler:run`** — Manually trigger a scheduled job.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-n` | `--name` | Job name |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

**`mapps scheduler:delete`** — Delete a scheduled job.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-n` | `--name` | Job name |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

```bash
# List all scheduled jobs
mapps scheduler:list -a ${MONDAY_APP_ID}

# Create a new scheduled job with retries and timeout
mapps scheduler:create -a ${MONDAY_APP_ID} -n "daily-cleanup" -s "0 0 * * *" -e "daily-cleanup" -r 3 -b 10 -t 60 -z us

# Create a simple job
mapps scheduler:create -a ${MONDAY_APP_ID} -s "0 * * * *" -e "my-endpoint" -n "hourly-sync" -d "Sync data every hour"

# Run a job on demand (don't wait for schedule)
mapps scheduler:run -a ${MONDAY_APP_ID} -n "daily-cleanup" -z us

# Delete a scheduled job
mapps scheduler:delete -a ${MONDAY_APP_ID} -n "daily-cleanup"
```

**Important:** Cron routes MUST use the path prefix `/mndy-cronjob/`. Max 5 jobs per region. IL region does not support cron. The `-e` flag specifies only the endpoint name (not the full path) — it becomes `/mndy-cronjob/<YOUR_ENDPOINT>`.

### 9. Security Reports

**`mapps code:report`** — Get security scan report for a monday-code deployment.

| Short | Long | Description |
|---|---|---|
| `-i` | `--appVersionId` | The app version ID |
| `-o` | `--output` | Save the full report to a JSON file |
| `-d` | `--outputDir` | Directory to save the report file (requires `-o`) |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

```bash
# View the security report in terminal
mapps code:report -i <version_id>

# Save report to a JSON file in a specific directory
mapps code:report -i <version_id> -o -d /path/to/directory

# Get report for a specific region
mapps code:report -i <version_id> -z us
```

### 10. Storage Data Export

**`mapps storage:export`** — Export all keys and values stored on monday storage api for a specific customer account.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-c` | `--clientAccountId` | Client account number |
| `-d` | `--fileDirectory` | File path to save export (optional) |
| `-f` | `--fileFormat` | Export format: `CSV` or `JSON` (default: `JSON`) |

```bash
# Export storage data as JSON
mapps storage:export -a ${MONDAY_APP_ID} -c <ACCOUNT_ID>

# Export as CSV to a specific directory
mapps storage:export -a ${MONDAY_APP_ID} -c <ACCOUNT_ID> -f CSV -d /path/to/export

# Export as JSON to a specific directory
mapps storage:export -a ${MONDAY_APP_ID} -c <ACCOUNT_ID> -f JSON -d /path/to/export
```

Or use MCP:
```
monday_apps_export_storage_data({ appId: APP_ID, appVersionId: VERSION_ID })
monday_apps_search_storage_records({ appId: APP_ID, appVersionId: VERSION_ID, term: "search_query" })
```

### 11. Version Management

**`mapps app:promote`** — Promote an app's draft version to live.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-i` | `--appVersionId` | The app version ID to promote |

**`mapps app:list`** — List all apps for the current user.

**`mapps manifest:export`** — Export app manifest.

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID (exports live version) |
| `-i` | `--appVersionId` | The app version ID |
| `-p` | `--manifestPath` | Path to export manifest files to |

```bash
# List all apps
mapps app:list

# List all versions
mapps app-version:list -i ${MONDAY_APP_ID}

# View builds for a version
mapps app-version:builds -i <version_id>

# List features for a version
mapps app-features:list -a ${MONDAY_APP_ID} -i <version_id>

# Promote draft to live
mapps app:promote -a ${MONDAY_APP_ID} -i <version_id>

# Export the app manifest
mapps manifest:export -a ${MONDAY_APP_ID} -p ./exports
```

Or use MCP:
```
monday_apps_get_app_versions({ appId: APP_ID })
monday_apps_promote_app({ appId: APP_ID, appVersionId: VERSION_ID })
```

## Troubleshooting Workflows

### "My app is returning errors in production"

1. Get the version ID: `mapps app-version:list -i ${MONDAY_APP_ID}`
2. Check deployment status: `mapps code:status -i <version_id>`
3. Stream logs to find errors: `mapps code:logs -i <version_id> -t console`
4. Search logs for error patterns: `mapps code:logs -i <version_id> -s live -r "error|exception|failed"`
5. Check HTTP error logs: `mapps code:logs -i <version_id> -t http`
6. Check if environment variables are set: `mapps code:env -i ${MONDAY_APP_ID} -m list-keys`
7. Check if secrets are set: `mapps code:secret -i ${MONDAY_APP_ID} -m list-keys`
8. If DB-related: get connection string and inspect data: `mapps database:connection-string -a ${MONDAY_APP_ID}`

### "My app is slow / timing out"

1. Check HTTP latency logs: `mapps code:logs -i <version_id> -t http`
2. Search for slow operations: `mapps code:logs -i <version_id> -s live -r "timeout|slow|ETIMEDOUT"`
3. Check alert board for latency alerts (see Section 7)
4. Review cron jobs that may be consuming resources: `mapps scheduler:list -a ${MONDAY_APP_ID}`
5. Check security report for dependency issues: `mapps code:report -i <version_id>`

### "Environment variable or secret not working"

1. Verify env var keys are set: `mapps code:env -i ${MONDAY_APP_ID} -m list-keys`
2. Verify secret keys are set: `mapps code:secret -i ${MONDAY_APP_ID} -m list-keys`
3. For multi-region apps, check the correct region: add `-z us` (or `eu`, `au`, `il`)
4. Re-deploy after setting env vars (they are baked in at deploy time)
5. In code, ensure you're using the right accessor:
   - Env vars: `EnvironmentVariablesManager` from `@mondaycom/apps-sdk`
   - Secrets: `SecretsManager` from `@mondaycom/apps-sdk`

### "MNDY_MONGODB_CONNECTION_STRING is undefined"

1. This is auto-injected after first deploy. Deploy an empty app first if needed.
2. Check if it's set: `mapps code:env -i ${MONDAY_APP_ID} -m list-keys`
3. Get it directly: `mapps database:connection-string -a ${MONDAY_APP_ID}`
4. For local dev: `docker run -d -p 27017:27017 mongo:7` and set manually in `.env`

### "Deployment failed or stuck"

1. Check status: `mapps code:status -i <version_id>`
2. Verify `mapps` is authenticated: `mapps --version` then `mapps app:list`
3. Verify app ID is correct: check Developer Center > General Settings
4. Check build locally first: `npm run build` (fix all TypeScript errors)
5. Ensure `node_modules/` is excluded from deployment artifacts

## Notes

- Always use MCP tools (`mcp__monday-apps__*`) when available for programmatic access
- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected - never set it manually
- Multi-region apps need per-region secret/env var configuration
- Alerts are global (apply to all regions), but cron jobs are per-region
- Logs require a version ID - get it via `mapps app-version:list` first
- Security scanning is non-blocking (deployment proceeds regardless of findings)
- After changing env vars or secrets, a re-deploy is required
- Use `--verbose` on any command for advanced debug logging
