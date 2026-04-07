# Error Taxonomy

Reference for monday.com API error codes, HTTP status codes, and handling strategies.

## Critical: HTTP 200 Contains Errors

The monday.com API returns **HTTP 200 for application-level GraphQL errors**. You MUST inspect the `errors` array in every response body, not just the HTTP status code.

## Error Response Format

```json
{
  "data": { ... },
  "errors": [
    {
      "message": "Human-readable description",
      "locations": [{"line": 1, "column": 5}],
      "path": ["boards", 0, "items_page"],
      "extensions": {
        "code": "ErrorCodeIdentifier",
        "status_code": 200,
        "error_data": {}
      }
    }
  ],
  "account_id": 12345
}
```

## Error Categories

### Authentication Errors (401)

| Error | Cause | Action |
|-------|-------|--------|
| 401 Unauthorized | Missing or invalid API token | Check token, do not add "Bearer" prefix |
| 401 IP restricted | Admin restricted API access to specific IPs | Contact account admin |

### Authorization Errors (403)

| Error | Cause | Action |
|-------|-------|--------|
| `UserUnauthorizedException` | User lacks permission to the resource | Token mirrors user's UI permissions — check user has access |
| `USER_ACCESS_DENIED` | User inactive, view-only, or unconfirmed email | Activate user or use a different token |
| `missingRequiredPermissions` | OAuth app scopes insufficient | Request additional scopes |

### Validation Errors (HTTP 200)

| Error | Cause | Action |
|-------|-------|--------|
| `ColumnValueException` | Wrong JSON format for column type | Use `get_column_type_info` to get correct format |
| `CorrectedValueException` | Value auto-corrected by API | Check the corrected value in response |
| `ItemNameTooLongException` | Name > 255 characters | Shorten the item name |
| `ItemsLimitationException` | Board exceeds 10,000 items | Archive old items or use a new board |
| Parse error | Malformed GraphQL query | Fix query syntax |

### Rate Limit Errors (429)

| Error | Cause | Action |
|-------|-------|--------|
| `COMPLEXITY_BUDGET_EXHAUSTED` | Complexity budget exceeded | Wait `reset_in_x_seconds`, reduce query complexity |
| `DAILY_LIMIT_EXCEEDED` | Daily call limit reached | Wait until midnight UTC, optimize call count |
| "Minute limit rate exceeded" | Per-minute call limit hit | Wait per `Retry-After` header |
| `maxConcurrencyExceeded` | Too many simultaneous requests | Queue requests, reduce parallelism |
| `IP_RATE_LIMIT_EXCEEDED` | IP-level rate limit | Reduce request rate from this IP |

### Conflict Errors (409/423)

| Error | Cause | Action |
|-------|-------|--------|
| `DeleteLastGroupException` | Tried to delete the only group on a board | Create a new group first |
| 423 Resource locked | Concurrent update in progress | Retry with short backoff (1-2s) |

### Server Errors (422/500)

| Error | Cause | Action |
|-------|-------|--------|
| `RecordInvalidException` (422) | Too many board subscribers or other constraint | Check the specific constraint |
| 500 Internal Server Error | Server-side issue | Retry with exponential backoff |

## Retry Decision Matrix

| HTTP Status | Error Code | Retry? | Strategy |
|-------------|-----------|--------|----------|
| 429 | `COMPLEXITY_BUDGET_EXHAUSTED` | Yes | Wait `reset_in_x_seconds` |
| 429 | `maxConcurrencyExceeded` | Yes | Short backoff (1-2s) |
| 429 | `IP_RATE_LIMIT_EXCEEDED` | Yes | Backoff (5-10s) |
| 200 | `DAILY_LIMIT_EXCEEDED` | No | Wait until midnight UTC |
| 200 | `ColumnValueException` | No | Fix the column value format |
| 200 | `ItemsLimitationException` | No | Reduce board size |
| 401 | — | No | Fix authentication |
| 403 | — | No | Fix permissions |
| 423 | — | Yes | Short backoff (1-2s) |
| 500 | — | Yes | Exponential backoff (max 3 retries) |
