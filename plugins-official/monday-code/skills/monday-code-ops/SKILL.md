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

Check the current deployment status and version info. See [references/deployment-status.md](references/deployment-status.md) for full CLI flags and MCP alternatives.

```bash
mapps app-version:list -i ${MONDAY_APP_ID}
mapps code:status -i <version_id>
```

### 2. Get Deployment URLs

**Backend (Serverless) URL:** Retrieve via `mapps code:status -i <version_id>` or the monday SDK context (`monday.get("context")` returns `appVersion.mondayCodeHostingUrl`).

**Frontend (CDN) URL:** Available in the Developer Center > App > Host on monday > Client-side code section.

See [references/deployment-status.md](references/deployment-status.md) for MCP alternatives.

### 3. View Logs

Stream or search production logs. See [references/logs.md](references/logs.md) for full CLI flags, date filtering, and regex search options.

```bash
mapps code:logs -i <version_id>                          # Stream live logs
mapps code:logs -i <version_id> -t http                  # HTTP request logs only
mapps code:logs -i <version_id> -t console               # Console/stdout only
mapps code:logs -i <version_id> -s live -r "error|timeout"  # Search with regex
```

### 4. Environment Variables

Manage environment variables. See [references/env-and-secrets.md](references/env-and-secrets.md) for full CLI flags and MCP alternatives.

```bash
mapps code:env -i ${MONDAY_APP_ID} -m list-keys           # List keys
mapps code:env -i ${MONDAY_APP_ID} -m set -k KEY -v "val" # Set a variable
```

Environment variables require a re-deploy to take effect.

### 5. Secrets Management

Manage secret variables. See [references/env-and-secrets.md](references/env-and-secrets.md) for full CLI flags, common secrets, and runtime access patterns.

```bash
mapps code:secret -i ${MONDAY_APP_ID} -m list-keys              # List keys
mapps code:secret -i ${MONDAY_APP_ID} -m set -k KEY -v "secret" # Set a secret
```

### 6. Database Connection String

Get the MongoDB connection string. See [references/database.md](references/database.md) for limits and important notes.

```bash
mapps database:connection-string -a ${MONDAY_APP_ID}
```

`MNDY_MONGODB_CONNECTION_STRING` is auto-injected after first deploy — never set it manually.

### 7. Alerts and Monitoring

monday code alerts auto-create a monday.com board for notifications. See [references/alerts-and-monitoring.md](references/alerts-and-monitoring.md) for detailed setup, alert types, and best practices.

3 alert types: HTTP error rate, HTTP latency response, and runtime limit quota. Set up via Developer Center > Host on monday > Server-side code > Alert policies tab.

Query the alerts board via MCP:
```
mcp__monday__get_board_items_page({ boardId: ALERT_BOARD_ID })
```

### 8. Cron Jobs Management

View and manage scheduled background jobs. See [references/cron-jobs.md](references/cron-jobs.md) for full CLI flags and constraints.

```bash
mapps scheduler:list -a ${MONDAY_APP_ID}                     # List jobs
mapps scheduler:create -a ${MONDAY_APP_ID} -n "name" -s "0 * * * *" -e "endpoint"  # Create
mapps scheduler:run -a ${MONDAY_APP_ID} -n "name"            # Run on demand
```

Cron routes MUST use `/mndy-cronjob/` prefix. Max 5 jobs per region. IL region does not support cron.

### 9. Security Reports

Get security scan report. See [references/security-and-storage.md](references/security-and-storage.md) for full CLI flags.

```bash
mapps code:report -i <version_id>
```

### 10. Storage Data Export

Export stored data for a customer account. See [references/security-and-storage.md](references/security-and-storage.md) for full CLI flags and MCP alternatives.

```bash
mapps storage:export -a ${MONDAY_APP_ID} -c <ACCOUNT_ID>
```

### 11. Version Management

Manage app versions, promotions, and manifests. See [references/version-management.md](references/version-management.md) for full CLI flags and MCP alternatives.

```bash
mapps app-version:list -i ${MONDAY_APP_ID}               # List versions
mapps app:promote -a ${MONDAY_APP_ID} -i <version_id>    # Promote to live
mapps manifest:export -a ${MONDAY_APP_ID} -p ./exports   # Export manifest
```

## Examples

**Example 1: Debug a failing production app**

User says: "My app is returning errors in production"

Actions:
1. Get the version ID: `mapps app-version:list -i ${MONDAY_APP_ID}`
2. Check deployment status: `mapps code:status -i <version_id>`
3. Stream console logs: `mapps code:logs -i <version_id> -t console`
4. Search for errors: `mapps code:logs -i <version_id> -s live -r "error|exception|failed"`
5. Check env vars are set: `mapps code:env -i ${MONDAY_APP_ID} -m list-keys`

Result: Found uncaught exception in webhook handler. Missing `MONDAY_SIGNING_SECRET` — set the secret and re-deployed to fix.

**Example 2: Get database access for debugging**

User says: "I need to look at the data in my production database"

Actions:
1. Get the connection string: `mapps database:connection-string -a ${MONDAY_APP_ID}`
2. Wait 1-2 minutes for the connection string to activate
3. Connect with MongoDB Compass or mongosh using the returned URI

Result: Provided connection string. User connected via MongoDB Compass and inspected the `tasks` collection.

**Example 3: Set up monitoring for a new deployment**

User says: "I just deployed, how do I monitor it?"

Actions:
1. Check deployment status: `mapps code:status -i <version_id>`
2. Guide user to set up alerts via Developer Center > Alert policies
3. Show how to query the alert board: `mcp__monday__get_board_items_page({ boardId: ALERT_BOARD_ID })`
4. Stream initial logs to verify healthy operation: `mapps code:logs -i <version_id>`

Result: Deployment confirmed healthy. Alert policies created for HTTP error rate (>5%) and latency (>2000ms at P95).

## Troubleshooting

### "My app is returning errors in production"

**Cause:** Application error in deployed code — could be missing env vars, unhandled exceptions, or dependency issues.
**Solution:** Follow Example 1 above. Check logs first (`-t console` for app errors, `-t http` for request errors), then verify env vars and secrets are set.

### "My app is slow / timing out"

**Cause:** Slow database queries, external API timeouts, or resource limits.
**Solution:** Check HTTP latency logs (`-t http`), search for timeout patterns (`-r "timeout|slow|ETIMEDOUT"`), review cron jobs consuming resources (`scheduler:list`), and check the security report for dependency issues.

### "Environment variable or secret not working"

**Cause:** Env vars require re-deploy, or wrong accessor used in code.
**Solution:** Verify keys are set (`code:env -m list-keys` / `code:secret -m list-keys`). For multi-region apps, check the correct region with `-z`. Re-deploy after changes. In code, use `EnvironmentVariablesManager` for env vars and `SecretsManager` for secrets (both from `@mondaycom/apps-sdk`).

### "MNDY_MONGODB_CONNECTION_STRING is undefined"

**Cause:** Auto-injected after first deploy — won't exist if the app has never been deployed.
**Solution:** Deploy at least once. Verify with `code:env -m list-keys`. Get it directly with `database:connection-string`. For local dev: `docker run -d -p 27017:27017 mongo:7` and set manually in `.env`.

### "Deployment failed or stuck"

**Cause:** Build errors, auth issues, or incorrect app ID.
**Solution:** Check status (`code:status`), verify auth (`mapps app:list`), verify app ID in Developer Center, build locally first (`npm run build`), ensure `node_modules/` is excluded.

## Notes

- Always use MCP tools (`mcp__monday-apps__*`) when available for programmatic access
- Multi-region apps need per-region secret/env var configuration
- Alerts are global (apply to all regions), but cron jobs are per-region
- Logs require a version ID — get it via `mapps app-version:list` first
- After changing env vars or secrets, a re-deploy is required
- Use `--verbose` on any command for advanced debug logging
