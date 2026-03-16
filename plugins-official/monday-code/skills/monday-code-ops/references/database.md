# Database Connection String Reference

Detailed CLI flags and notes for accessing the monday-code MongoDB database.

## `mapps database:connection-string` — Get the connection string for your app database

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

## Important Notes

- It takes about 1-2 minutes after running the command for the connection string to actually become active and usable
- `MNDY_MONGODB_CONNECTION_STRING` is auto-injected by monday-code after first deploy — never set it manually
- Max storage: 1 GiB per app
- Limits: 50,000 reads / 20,000 writes per region per day
- The connection string can be used with MongoDB Compass, mongosh, or other MongoDB tools for direct database inspection
