---
name: ops
description: Production operations guidance for the monday.com API. Covers error handling, rate limits, complexity budgeting, retry strategies, webhooks, and caching. Use when preparing an API integration for production or debugging production issues.
argument-hint: "<topic, e.g. 'rate limits' or 'error handling' or 'production checklist'>"
user-invocable: true
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, mcp__monday__all_monday_api, mcp__monday__get_graphql_schema]
---

# API Operations

Production operations guidance for the monday.com GraphQL API. Covers error handling, rate limits, complexity budgeting, retry strategies, webhooks, caching, and a production readiness checklist.

## Scope

- Operational guidance for production API usage
- Does NOT cover writing queries (use `/monday-api:build-query`)
- Does NOT cover project setup (use `/monday-api:setup`)

---

## Workflow

### Step 1: Understand the Request

| User says | Action |
|-----------|--------|
| "how do I handle errors" / "error handling" | Go to Flow 1 |
| "rate limits" / "I'm being throttled" / "429" | Go to Flow 2 |
| "complexity" / "query too complex" | Go to Flow 3 |
| "retry" / "transient errors" / "backoff" | Go to Flow 4 |
| "webhooks" / "real-time" / "polling" | Go to Flow 5 |
| "caching" / "performance" / "too many calls" | Go to Flow 6 |
| "production checklist" / "go live" / "production ready" | Go to Flow 7 |

---

### Flow 1: Error Handling

monday.com returns **HTTP 200 for application-level errors**. You MUST check the `errors` array in every response.

**Response structure:**
```json
{
  "data": { "boards": [...] },
  "errors": [
    {
      "message": "Description",
      "locations": [{"line": 1, "column": 5}],
      "path": ["boards"],
      "extensions": {
        "code": "ErrorCode",
        "status_code": 200
      }
    }
  ],
  "account_id": 12345
}
```

**Partial responses are possible** — you can get BOTH `data` and `errors` in the same response. Some fields succeed while others fail.

**Error handling boilerplate:**

```typescript
const result = await client.request(query, variables);

// With @mondaydotcomorg/api SDK — use rawRequest for error access
const { data, errors, extensions } = await client.rawRequest(query, variables);

if (errors?.length) {
  for (const error of errors) {
    const code = error.extensions?.code;

    if (code === 'COMPLEXITY_BUDGET_EXHAUSTED') {
      // Wait and retry
      const retryAfter = error.extensions?.retry_in_seconds || 60;
      await sleep(retryAfter * 1000);
    } else if (code === 'UserUnauthorizedException') {
      // Token lacks permission — cannot retry
      throw new Error(`Permission denied: ${error.message}`);
    } else {
      console.error(`API error [${code}]: ${error.message}`);
    }
  }
}

// Always check data even if errors exist (partial response)
if (data) {
  // Process partial or full data
}
```

For the full error taxonomy, read `docs/error-taxonomy.md` in this plugin directory.

---

### Flow 2: Rate Limits

monday.com has **4 independent rate limit axes** that can all trigger simultaneously:

| Axis | Limit | Scope | Error |
|------|-------|-------|-------|
| **Complexity** | 5M-10M points/min (by plan) | Per-account, sliding window | `COMPLEXITY_BUDGET_EXHAUSTED` (429) |
| **Daily calls** | 200-25,000/day (by plan) | Per-app, resets midnight UTC | `DAILY_LIMIT_EXCEEDED` |
| **Per-minute** | 1,000-5,000/min (by plan) | Per-app | "Minute limit rate exceeded" + `Retry-After` header |
| **Concurrency** | 40-250 simultaneous (by plan) | Per-account | `maxConcurrencyExceeded` (429) |

Plus **IP rate limit**: 5,000 requests per 10 seconds per IP.

**Critical:** ALL requests count toward limits — including failed requests, error responses, and retries.

**Plan-based limits:**

| Plan | Daily Calls | Per-Minute | Concurrency | Complexity/min |
|------|-------------|------------|-------------|----------------|
| Free/Trial | 200 | 1,000 | 40 | 1,000,000 |
| Standard | 1,000 | 1,000 | 40 | 5,000,000 |
| Pro | 10,000 | 2,500 | 100 | 5,000,000 |
| Enterprise | 25,000 | 5,000 | 250 | 10,000,000 |

For complete rate limit details, read `docs/rate-limits-reference.md` in this plugin directory.

---

### Flow 3: Complexity Budgeting

Every query has a complexity cost. Query it alongside your data:

```graphql
query {
  complexity {
    before
    query
    after
    reset_in_x_seconds
  }
  boards(ids: [BOARD_ID]) {
    items_page(limit: 50) {
      items { id name }
    }
  }
}
```

**Offer to demo:** Run a complexity-only query right now to show the developer their budget:

Call `mcp__monday__all_monday_api` with:
```graphql
query {
  complexity {
    before
    query
    after
    reset_in_x_seconds
  }
}
```
(variables: `{}`)

Show the result — `before` is the remaining budget, `query` is the cost of this call (very low), `after` is the remaining budget after. `reset_in_x_seconds` tells when the window resets.

**Reducing complexity:**
- Request only needed fields (don't fetch `column_values` if you only need `id` and `name`)
- Use smaller pagination limits
- Avoid deeply nested queries
- Use `items_page` with filtering instead of fetching all items and filtering client-side
- Complexity-only queries (just the `complexity` field) cost only 0.1 daily calls

---

### Flow 4: Retry Strategy

**Exponential backoff with jitter:**

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelayMs = 1000
): Promise<T> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries) throw error;

      // Check for Retry-After header
      const retryAfter = error.response?.headers?.['retry-after'];
      const delay = retryAfter
        ? parseInt(retryAfter) * 1000
        : baseDelayMs * Math.pow(2, attempt) + Math.random() * 1000;

      console.log(`Retry ${attempt + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Unreachable');
}
```

**Which errors to retry:**

| Error | Retry? | Strategy |
|-------|--------|----------|
| Rate limit (429) | Yes | Use `Retry-After` or `retry_in_seconds` |
| Complexity exceeded | Yes | Wait `reset_in_x_seconds`, then retry |
| Concurrency limit | Yes | Short backoff (1-2s) |
| 500 Internal Server Error | Yes | Exponential backoff |
| 423 Resource Locked | Yes | Short backoff (1-2s) |
| Network timeout | Yes | Exponential backoff |
| 401 Unauthorized | No | Fix token |
| 403 Forbidden | No | Fix permissions |
| Parse error / validation | No | Fix query |
| `ColumnValueException` | No | Fix column value format |

---

### Flow 5: Webhooks vs Polling

**Use webhooks when:**
- You need real-time notifications of changes
- You want to react to item creation, status changes, column updates
- You want to reduce API call consumption

**Use polling when:**
- One-time data sync or batch processing
- Development and testing
- Webhook delivery is unreliable for your infrastructure

**Webhook event types (23+):** `create_item`, `change_column_value`, `change_status_column_value`, `change_name`, `create_update`, `delete_item`, `archive_item`, `move_item_to_group`, and more.

**Webhook setup:**
```graphql
mutation {
  create_webhook(
    board_id: BOARD_ID
    url: "https://your-server.com/webhook"
    event: change_column_value
  ) {
    id
    board_id
  }
}
```

**Challenge/response verification:** monday.com sends a challenge request on creation. Your endpoint must respond with the challenge token:

```json
// Incoming POST from monday.com
{ "challenge": "abc123..." }

// Your response (must return within 5 seconds)
{ "challenge": "abc123..." }
```

**Webhook payloads include JWT auth** when using app tokens — verify with your app's Signing Secret.

---

### Flow 6: Caching Strategies

**What to cache (changes rarely):**

| Data | Recommended TTL | Why |
|------|----------------|-----|
| Board structure (columns, groups) | 1 hour | Changes when board is reconfigured |
| Column definitions (types, settings) | 1 hour | Changes when columns are added/modified |
| Users and teams | 15 minutes | Changes when team composition changes |
| Workspace list | 1 hour | Rarely changes |

**What NOT to cache:**

| Data | Why |
|------|-----|
| Item values | Changes frequently via UI and API |
| Column values | Updated by multiple users in real-time |
| Activity logs | Always growing |
| Board item counts | Changes with every create/delete |

**Caching pattern:**

```typescript
const cache = new Map<string, { data: any; expiresAt: number }>();

async function cachedQuery<T>(key: string, ttlMs: number, query: () => Promise<T>): Promise<T> {
  const cached = cache.get(key);
  if (cached && cached.expiresAt > Date.now()) {
    return cached.data as T;
  }

  const data = await query();
  cache.set(key, { data, expiresAt: Date.now() + ttlMs });
  return data;
}

// Usage
const columns = await cachedQuery(
  `board-${boardId}-columns`,
  60 * 60 * 1000, // 1 hour
  () => client.request(`{ boards(ids: [${boardId}]) { columns { id title type } } }`)
);
```

---

### Flow 7: Production Checklist

Present this checklist when a developer asks about production readiness:

## Production Readiness Checklist

- [ ] **Pin API version** — set explicit `apiVersion: '2026-01'`, never rely on default
- [ ] **Handle errors** — check `response.errors` array (HTTP 200 can contain errors)
- [ ] **Handle partial responses** — `data` and `errors` can coexist
- [ ] **Implement retry with backoff** — exponential backoff + jitter for rate limits and 5xx
- [ ] **Respect Retry-After** — use the header/field value, don't guess
- [ ] **Monitor complexity** — query `complexity` field periodically to track budget usage
- [ ] **Use app tokens in production** — not personal API tokens (which are tied to one user)
- [ ] **Add rate limit awareness** — track daily call usage, back off before hitting limits
- [ ] **Cache stable data** — board structure, columns, users (1h TTL)
- [ ] **Use webhooks over polling** — for real-time change detection
- [ ] **Log API calls** — request/response for debugging production issues
- [ ] **Test against RC** — validate queries against the Release Candidate before it becomes Current
- [ ] **Request minimal fields** — only fetch what you use to reduce complexity cost
- [ ] **Use pagination** — `items_page` with appropriate limits, never fetch unbounded

If the developer wants to verify specific items, offer to test their queries or check their error handling code.

---

## Error Handling

| Error | Action |
|-------|--------|
| User asks about specific error code | Look up in `docs/error-taxonomy.md` |
| Rate limit hit during demo | Show how to read headers, suggest backoff |
| General "my app is slow" | Start with complexity analysis (Flow 3), then caching (Flow 6) |
| "How do I monitor API usage" | Show complexity query and suggest the `platform_api` query for usage analytics |
| MCP not connected | Tell user to check `/mcp` |
