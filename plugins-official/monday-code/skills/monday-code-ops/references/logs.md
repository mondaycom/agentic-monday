# Logs Reference

Detailed CLI flags and examples for streaming and searching production logs.

## `mapps code:logs` — Stream logs from a monday code deployment

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

## Examples

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

## Getting the version ID

```bash
mapps app-version:list -i ${MONDAY_APP_ID}
```

Or use MCP:
```
monday_apps_get_app_versions({ appId: APP_ID })
```
