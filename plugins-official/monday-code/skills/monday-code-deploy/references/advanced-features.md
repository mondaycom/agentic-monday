# Advanced Features Reference

Detailed documentation for advanced monday-code deployment features.

## Multi-Region Deployment

monday-code supports 4 regions: **us, eu, au, il**.

**Important constraints:**
- Multi-region must be enabled BEFORE using Document DB (irreversible)
- Public apps only (not private/dev apps)
- Environment variables are set per-region
- Alerts are global (apply to all regions)
- Cron jobs are per-region (IL not supported for cron)

Enable multi-region in the Developer Center before first production deploy.

**Setting secrets per region:**
```bash
mapps secrets:set -a ${MONDAY_APP_ID} -k KEY -v "value" -z us
mapps secrets:set -a ${MONDAY_APP_ID} -k KEY -v "value" -z il
```

**Listing secrets per region:**
```bash
mapps secrets:list -a ${MONDAY_APP_ID} -z us
mapps secrets:list -a ${MONDAY_APP_ID} -z il
```

## Cron Jobs

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

## Alerts

Monitor your app with automated alerts (creates a monday.com board):

**3 alert types:**
1. **HTTP error rate** - Triggers when error rate exceeds threshold
2. **HTTP latency** - Triggers when response time exceeds threshold
3. **Runtime limit quota** - Triggers when approaching compute limits

**Create alerts via Developer Center** > App > Alerts section.

Alerts are global (apply to all regions). They auto-create a monday.com board for notifications.

## Promoting App Versions

Promote a development version to live:

```
monday_apps_promote_app({ appId: APP_ID, appVersionId: VERSION_ID })
```

Or through the Developer Center UI.

**When to promote:**
- After verifying deployment with `mapps code:status`
- After setting all required environment variables and secrets
- After configuring OAuth scopes in the Developer Center
- After running and passing security scans
