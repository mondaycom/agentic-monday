# Logs Reference

Detailed CLI flags and examples for streaming live logs and fetching historical production logs.

## `mapps code:logs` — Stream logs from a monday code deployment

| Short | Long | Description |
|---|---|---|
| `-i` | `--appVersionId` | The app version ID |
| `-t` | `--logsType` | Log type: `http` for HTTP events, `console` for stdout |
| `-s` | `--eventSource` | Source: `live` for live events, `history` for past events |
| `-f` | `--logsStartDate` | Start date `MM/DD/YYYY HH:mm` (only with `-s history`) |
| `-e` | `--logsEndDate` | End date `MM/DD/YYYY HH:mm` (only with `-s history`) |
| `-r` | `--logSearchFromText` | Text search filter passed to historical log queries (only with `-s history`) |
| `-z` | `--region` | Region: `us`, `eu`, `au`, or `il` |

Global flags: `--verbose` (advanced logs), `--print-command` (print executed command).

For non-interactive agent usage, pass both `-s` and `-t`. Otherwise the CLI may prompt for source or log type.

## Examples

```bash
# Stream live console logs
mapps code:logs -i <version_id> -s live -t console

# Stream live HTTP request logs
mapps code:logs -i <version_id> -s live -t http

# Stream live console logs and filter locally
mapps code:logs -i <version_id> -s live -t console | rg "error|timeout"

# View historical console logs within a date range
mapps code:logs -i <version_id> -s history -t console -f "03/15/2026 00:00" -e "03/16/2026 23:59" -r ""

# Search historical console logs for text
mapps code:logs -i <version_id> -s history -t console -f "03/15/2026 00:00" -e "03/16/2026 23:59" -r "error|timeout"

# View historical logs
mapps code:logs -i <version_id> -s history -t console -f "03/15/2026 00:00" -e "03/16/2026 23:59" -r ""

# View logs for a specific region
mapps code:logs -i <version_id> -s live -t console -z eu

# Combine: search console logs in EU region within a date range
mapps code:logs -i <version_id> -s history -t console -f "03/15/2026 09:00" -e "03/15/2026 17:00" -r "uncaught" -z eu
```

## Getting the version ID

```bash
mapps app-version:list -i ${MONDAY_APP_ID}
```

Or use MCP:
```
monday_apps_get_app_versions({ appId: APP_ID })
```
