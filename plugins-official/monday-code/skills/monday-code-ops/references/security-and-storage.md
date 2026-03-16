# Security Reports & Storage Export Reference

## Security Reports

### `mapps code:report` — Get security scan report for a monday-code deployment

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

Security scanning is non-blocking — deployment proceeds regardless of findings.

## Storage Data Export

### `mapps storage:export` — Export all keys and values stored on monday storage API for a specific customer account

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

### MCP Alternatives

```
monday_apps_export_storage_data({ appId: APP_ID, appVersionId: VERSION_ID })
monday_apps_search_storage_records({ appId: APP_ID, appVersionId: VERSION_ID, term: "search_query" })
```
