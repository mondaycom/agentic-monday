# Deployment Status Reference

Detailed CLI and MCP commands for checking deployment status, versions, builds, and features.

## CLI Commands

### `mapps code:status` — Status of a specific project hosted on monday-code

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

### `mapps app-version:list` — List all versions for a specific app

| Short | Long | Description |
|---|---|---|
| `-i` | `--appId` | The app ID |

### `mapps app-version:builds` — List all builds for a specific app version

| Short | Long | Description |
|---|---|---|
| `-i` | `--appVersionId` | The app version ID |

### `mapps app-features:list` — List all features for a specific app version

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-i` | `--appVersionId` | The app version ID |

```bash
mapps app-features:list -a ${MONDAY_APP_ID} -i <version_id>
```

## MCP Alternatives

```
monday_apps_get_deployment_status({ appVersionId: VERSION_ID })
monday_apps_get_app_versions({ appId: APP_ID })
monday_apps_get_app_version({ appId: APP_ID, versionId: VERSION_ID })
monday_apps_get_app_features({ appId: APP_ID })
```
