# Rate Limits Reference

Complete reference for monday.com API rate limits.

## Four Rate Limit Axes

monday.com enforces 4 independent rate limits simultaneously. Hitting any one of them will throttle your requests.

### 1. Complexity Budget (per sliding minute window)

The complexity budget limits how "expensive" your queries can be within a 60-second sliding window.

| Token Type | Read Budget | Write Budget |
|------------|-------------|--------------|
| App tokens | 5,000,000/min | 5,000,000/min (separate) |
| Personal API tokens | 10,000,000/min combined | (same budget for reads+writes) |
| Trial/Free/NGO | 1,000,000/min | (same) |

- Single query hard cap: 5,000,000 complexity points per call
- Sliding window resets 60 seconds after the first call
- Error: `ComplexityException` / HTTP 429 `COMPLEXITY_BUDGET_EXHAUSTED`
- App tokens have SEPARATE read and write budgets

### 2. Daily Call Limit (resets midnight UTC)

| Plan | Daily Calls |
|------|-------------|
| Free/Trial | 200 |
| Standard/Basic | 1,000 |
| Pro | 10,000 (soft limit) |
| Enterprise | 25,000 (soft limit) |

- **Failed calls still count** toward the daily limit
- Rate-limit errors cost 0.1 calls
- Complexity-only queries cost 0.1 calls
- Error: `DAILY_LIMIT_EXCEEDED`

### 3. Per-Minute Call Limit

| Plan | Calls/Minute |
|------|-------------|
| Enterprise | 5,000 |
| Pro | 2,500 |
| Other | 1,000 |

**Endpoint-specific limits:**

| Operation | Limit |
|-----------|-------|
| Create/duplicate board | 40/min |
| Duplicate group | 40/min |
| Connect project to portfolio | 15/min |
| Items query | 100 items max |
| App subscriptions query | 120/min |
| `display_value` on FormulaValue | 10,000 values/min (max 5 formula columns/request) |

- Error: "Minute limit rate exceeded" + `Retry-After` header

### 4. Concurrency Limit

| Plan | Max Simultaneous Requests |
|------|--------------------------|
| Enterprise | 250 |
| Pro | 100 |
| Other | 40 |

- Error: `Concurrency limit exceeded` / HTTP 429 `maxConcurrencyExceeded`

### 5. IP Rate Limit

5,000 requests per 10 seconds per source IP address.
- Error: `IP_RATE_LIMIT_EXCEEDED`

## Checking Your Budget

Query complexity alongside your data:
```graphql
query {
  complexity {
    before         # remaining budget before this query
    query          # cost of this specific query
    after          # remaining budget after
    reset_in_x_seconds  # seconds until budget resets
  }
  boards(ids: [BOARD_ID]) { ... }
}
```

Complexity-only queries (just the `complexity` field) cost only 0.1 daily calls — use for budget monitoring.

## Response Headers

Rate limit information is returned in response headers:

| Header | Meaning |
|--------|---------|
| `Retry-After` | Seconds to wait before retrying (on per-minute limit) |

## Key Rules

1. **All requests count** — including failed requests, error responses, and retries
2. **All 4 limits are independent** — you can hit any combination simultaneously
3. **Failed calls consume quota** — even a malformed query costs a daily call
4. **Rate limit errors themselves cost 0.1 daily calls** — aggressive retrying makes things worse
