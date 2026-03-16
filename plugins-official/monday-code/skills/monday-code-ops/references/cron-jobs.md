# Cron Jobs Reference

Detailed CLI flags and examples for managing scheduled background jobs.

## `mapps scheduler:list` — List all scheduled jobs

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

## `mapps scheduler:create` — Create a new scheduled job

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

## `mapps scheduler:run` — Manually trigger a scheduled job

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-n` | `--name` | Job name |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

## `mapps scheduler:delete` — Delete a scheduled job

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-n` | `--name` | Job name |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

## Examples

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

## Important Constraints

- Cron routes MUST use the path prefix `/mndy-cronjob/`
- Max 5 jobs per region
- IL region does not support cron
- The `-e` flag specifies only the endpoint name (not the full path) — it becomes `/mndy-cronjob/<YOUR_ENDPOINT>`
