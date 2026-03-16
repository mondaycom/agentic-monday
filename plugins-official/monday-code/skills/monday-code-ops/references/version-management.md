# Version Management Reference

Detailed CLI flags and examples for managing app versions, promotions, and manifests.

## `mapps app:promote` — Promote an app's draft version to live

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID |
| `-i` | `--appVersionId` | The app version ID to promote |

## `mapps app:list` — List all apps for the current user

No required flags.

## `mapps manifest:export` — Export app manifest

| Short | Long | Description |
|---|---|---|
| `-a` | `--appId` | The app ID (exports live version) |
| `-i` | `--appVersionId` | The app version ID |
| `-p` | `--manifestPath` | Path to export manifest files to |

## Examples

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

## MCP Alternatives

```
monday_apps_get_app_versions({ appId: APP_ID })
monday_apps_promote_app({ appId: APP_ID, appVersionId: VERSION_ID })
```
