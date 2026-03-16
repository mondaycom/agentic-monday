# Environment Variables & Secrets Reference

Detailed CLI flags and examples for managing environment variables and secrets.

## Environment Variables

### `mapps code:env` — Manage environment variables for an app hosted on monday-code

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

### MCP Alternatives

```
monday_apps_list_environment_variable_keys({ appId: APP_ID })
monday_apps_set_environment_variable({ appId: APP_ID, key: "KEY", value: "value" })
```

**Important:** Environment variables require a re-deploy to take effect.

## Secrets

### `mapps code:secret` — Manage secret variables for an app hosted on monday-code

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

### Common Secrets

- `MONDAY_CLIENT_SECRET` - JWT auth for fullstack apps (from Developer Center > OAuth)
- `MONDAY_SIGNING_SECRET` - Webhook verification for automations

### Accessing Secrets at Runtime

```typescript
import { SecretsManager } from "@mondaycom/apps-sdk";
const secrets = new SecretsManager();
const { value } = await secrets.getSecret("MONDAY_CLIENT_SECRET");
```
