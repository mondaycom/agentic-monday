# Pagination Patterns

Reference for paginating through items in the monday.com GraphQL API.

## The Two-Phase Pattern

monday.com uses cursor-based pagination with two separate queries:

1. **`items_page`** — nested inside `boards` or `groups`, returns the first page + a cursor
2. **`next_items_page`** — called at the **root level**, returns subsequent pages using the cursor

> **CRITICAL:** `next_items_page` MUST be a root-level query. Nesting it inside `boards {}` is the single most common pagination bug.

## Phase 1: Initial Query

```graphql
query {
  boards(ids: [BOARD_ID]) {
    items_page(limit: 100) {
      cursor
      items {
        id
        name
        column_values { id text value }
      }
    }
  }
}
```

### Parameters

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| `limit` | Int | 25 | 500 | Items per page |
| `query_params` | Object | — | — | Filter/sort rules (see Filtering below) |

## Phase 2: Subsequent Pages

```graphql
query ($cursor: String!) {
  next_items_page(limit: 100, cursor: $cursor) {
    cursor
    items {
      id
      name
      column_values { id text value }
    }
  }
}
```

- When `cursor` is `null`, there are no more pages
- Use the **same field selection** as Phase 1 for consistent results
- `next_items_page` is a **root-level query** — NOT nested inside boards

## Why next_items_page Is Root-Level

```
WRONG:
query {
  boards(ids: [123]) {
    next_items_page(cursor: "abc") {  # ERROR - not a field on Board type
      ...
    }
  }
}

CORRECT:
query {
  next_items_page(cursor: "abc", limit: 100) {
    cursor
    items { id name }
  }
}
```

The cursor already encodes which board and filters to use, so no board context is needed.

## Filtering with query_params

Apply filters on the first `items_page` call only. The cursor carries the filters forward.

```graphql
query {
  boards(ids: [BOARD_ID]) {
    items_page(limit: 100, query_params: {
      rules: [
        { column_id: "status", compare_value: ["Done", "Working on it"] },
        { column_id: "date4", compare_value: ["2026-01-01", "2026-12-31"], operator: between }
      ],
      operator: and
    }) {
      cursor
      items { id name }
    }
  }
}
```

### Rules

- `query_params` and `cursor` **cannot** be used in the same request
- Use `query_params` only on the initial `items_page` call
- The cursor on subsequent `next_items_page` calls inherits the filters
- `operator` at the top level: `and` (default) or `or`

## Full Pagination Loop

### TypeScript with @mondaydotcomorg/api SDK

```typescript
import { ApiClient } from '@mondaydotcomorg/api';

const client = new ApiClient({
  token: process.env.MONDAY_API_TOKEN,
  apiVersion: '2026-01'
});

async function getAllItems(boardId: string) {
  const allItems: any[] = [];

  // Phase 1: Initial query
  const firstPage = await client.request(`query ($boardId: [ID!]!) {
    boards(ids: $boardId) {
      items_page(limit: 500) {
        cursor
        items { id name column_values { id text value } }
      }
    }
  }`, { boardId: [boardId] });

  const page = firstPage.boards[0].items_page;
  allItems.push(...page.items);
  let cursor = page.cursor;

  // Phase 2: Subsequent pages (root-level!)
  while (cursor) {
    const nextPage = await client.request(`query ($cursor: String!) {
      next_items_page(limit: 500, cursor: $cursor) {
        cursor
        items { id name column_values { id text value } }
      }
    }`, { cursor });

    allItems.push(...nextPage.next_items_page.items);
    cursor = nextPage.next_items_page.cursor;
  }

  return allItems;
}
```

### JavaScript with fetch

```javascript
async function getAllItems(boardId, token) {
  const allItems = [];
  const endpoint = 'https://api.monday.com/v2';
  const headers = {
    'Authorization': token,
    'Content-Type': 'application/json',
    'API-Version': '2026-01'
  };

  // Phase 1
  let res = await fetch(endpoint, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      query: `{ boards(ids: [${boardId}]) { items_page(limit: 500) { cursor items { id name } } } }`
    })
  });
  let data = (await res.json()).data;
  allItems.push(...data.boards[0].items_page.items);
  let cursor = data.boards[0].items_page.cursor;

  // Phase 2
  while (cursor) {
    res = await fetch(endpoint, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        query: `query ($cursor: String!) { next_items_page(limit: 500, cursor: $cursor) { cursor items { id name } } }`,
        variables: { cursor }
      })
    });
    data = (await res.json()).data;
    allItems.push(...data.next_items_page.items);
    cursor = data.next_items_page.cursor;
  }

  return allItems;
}
```

## Pagination by Group

You can also paginate within a specific group:

```graphql
query {
  boards(ids: [BOARD_ID]) {
    groups(ids: ["GROUP_ID"]) {
      items_page(limit: 100) {
        cursor
        items { id name }
      }
    }
  }
}
```

Subsequent pages still use `next_items_page` at root level with the returned cursor.

## Performance Considerations

| Limit | Trade-off |
|-------|-----------|
| 25 (default) | Lowest complexity cost, many round trips |
| 100 | Good balance for most use cases |
| 500 (max) | Fewest round trips, highest complexity per call |

- Boards have a hard limit of **10,000 items**. Exceeding this throws `ItemsLimitationException`.
- Each page request counts against your daily API call limit.
- Request only the fields you need — `column_values` significantly increases complexity.

## items vs items_page

| Feature | `items` (legacy) | `items_page` (recommended) |
|---------|-------------------|---------------------------|
| Max items | 100 per call | 500 per page |
| Pagination | Offset-based (`page` param) | Cursor-based |
| Filtering | Limited | Full `query_params` support |
| Use case | Lookup by known item IDs | Fetch all/filtered board items |
| Nesting | On `boards` type | On `boards` type (first call) |

**Always prefer `items_page`** for fetching board items. Use `items(ids: [...])` only when you already have specific item IDs.
