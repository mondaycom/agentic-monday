---
name: build-query
description: Build, test, and debug monday.com GraphQL queries and mutations. Introspects the live schema, handles pagination, column value formatting, and validates queries by executing them. Use when writing API queries, debugging response shapes, or figuring out column value JSON formats.
argument-hint: "<what you want to do, e.g. 'create an item with status and date columns'>"
user-invocable: true
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, mcp__monday__get_graphql_schema, mcp__monday__get_type_details, mcp__monday__get_column_type_info, mcp__monday__all_monday_api, mcp__monday__get_board_info, mcp__monday__get_board_items_page]
---

# Build Query

Build, live-test, and debug monday.com GraphQL queries and mutations. Introspects the actual schema to ensure correctness, formats column values properly, and validates every query by executing it.

## Scope

- Helps external developers write correct GraphQL for the monday.com public API
- Complements the monday MCP tools — uses them for schema introspection and live testing
- Does NOT cover SDK setup (use `/monday-api:setup`) or production operations (use `/monday-api:ops`)

## Prerequisites

- The **monday** MCP server must be connected (check `/mcp`)
- A valid API token must be configured in the MCP server

---

## Workflow

### Step 1: Understand the Request

| User says | Action |
|-----------|--------|
| "get items from board X" / "read board data" | Go to Flow 1 (Read query) |
| "create an item" / "add a row" | Go to Flow 2 (Write mutation) |
| "update status column" / "set date value" | Go to Flow 3 (Column value write) |
| "get all items" / "paginate through board" | Go to Flow 4 (Pagination) |
| "my query returns an error" / "this isn't working" | Go to Flow 5 (Debug) |
| General "how do I query X" | Introspect schema first, then choose the right flow |

If the user hasn't specified a board ID, ask for one — most queries require it.

---

### Flow 1: Build a Read Query

**Step 1: Introspect the schema**

Use `mcp__monday__get_graphql_schema` with `operationType: "read"` to find available query fields. The output lists root-level queries (e.g., `boards`, `items`, `users`, `me`) and their argument types — use this to find the right query entry point.

Then use `mcp__monday__get_type_details` with the return type name (e.g., `Board`, `Item`) to discover available fields. The output lists field names and their types — only request fields the developer actually needs, as every field adds to complexity cost.

**Step 2: Build the query**

Construct the GraphQL query with proper arguments and field selection.

Rules:
- For board items, ALWAYS use `items_page` (not the deprecated `items` field on boards)
- Use `ids` parameter as an array: `boards(ids: [BOARD_ID])`
- Keep field selection minimal — only what's needed
- Use variables for dynamic values (not string interpolation)

**Step 3: Live-test the query**

Execute via `mcp__monday__all_monday_api` with the constructed query.

Show the actual response to the developer so they can see the exact shape.

**Step 4: Present working code**

Show the tested query wrapped in SDK code. Define the query in a `.graphql.ts` file and use code-generated types:

```typescript
// src/queries.graphql.ts
export const GET_BOARD_ITEMS = `
  query GetBoardItems($boardId: [ID!]!) {
    boards(ids: $boardId) {
      name
      items_page(limit: 25) {
        cursor
        items {
          id
          name
          column_values { id text value }
        }
      }
    }
  }
`;
```

```typescript
// src/index.ts
import 'dotenv/config';
import { ApiClient } from '@mondaydotcomorg/api';
import { GET_BOARD_ITEMS } from './queries.graphql';
import type { GetBoardItemsQuery } from './generated/graphql'; // from npm run codegen

const client = new ApiClient({ token: process.env.MONDAY_API_TOKEN! });

const result = await client.request<GetBoardItemsQuery>(GET_BOARD_ITEMS, { boardId: [BOARD_ID] });
```

---

### Flow 2: Build a Write Mutation

**Step 1: Introspect mutation arguments**

Use `mcp__monday__get_graphql_schema` with `operationType: "write"` to find the mutation.

Use `mcp__monday__get_type_details` to inspect required and optional arguments.

**Step 2: If column values are involved, get the exact JSON format**

Use `mcp__monday__get_column_type_info` for each column type being written. See Flow 3 for details.

The `column_values` argument takes a **JSON string** — this is the #1 source of developer errors. The JSON must be stringified and passed as a string value.

**Step 3: Build the mutation with variables**

Always use GraphQL variables for dynamic values:

```graphql
mutation ($boardId: ID!, $groupId: String!, $itemName: String!, $columnValues: JSON) {
  create_item(
    board_id: $boardId
    group_id: $groupId
    item_name: $itemName
    column_values: $columnValues
  ) {
    id
    name
  }
}
```

**Step 4: Confirm and test with test data**

Show the complete mutation and variables to the developer. State clearly: **"This mutation will write data. Shall I run it now to verify it works?"**

If they confirm, execute via `mcp__monday__all_monday_api`. If the mutation creates an item or makes a change, that's expected — the goal is to verify the query structure is correct. Note any test items created so the developer can clean them up.

**Step 5: Present working code**

Show the tested mutation wrapped in SDK code with proper variable handling.

---

### Flow 3: Column Value Formatting

This is where most developers get stuck. Each column type has a specific JSON format for writes.

**Step 1: Identify column types needed**

Ask which columns the developer wants to set, or inspect the board:

Use `mcp__monday__get_board_info` with the board ID to see column definitions.

The column IDs returned (e.g., `status`, `date4`, `numbers0`) are the keys to use in `column_values`. They are NOT the display names shown in the UI.

Example response:
```json
{
  "columns": [
    { "id": "status", "title": "Status", "type": "status" },
    { "id": "date4", "title": "Due Date", "type": "date" },
    { "id": "numbers0", "title": "Budget", "type": "numbers" }
  ]
}
```

**Step 2: Get the exact JSON format**

Use `mcp__monday__get_column_type_info` for each column type. This returns the exact schema.

**Step 3: Show the format with examples**

Common column value formats (quick reference):

| Type | Write JSON | Example |
|------|-----------|---------|
| status | `{"index": N}` or `{"label": "Label"}` | `{"label": "Done"}` |
| date | `{"date": "YYYY-MM-DD"}` | `{"date": "2026-01-15"}` |
| date + time | `{"date": "YYYY-MM-DD", "time": "HH:MM:SS"}` | `{"date": "2026-01-15", "time": "09:00:00"}` |
| numbers | `"42"` (number as string) | `"42"` |
| text | `"hello"` (plain string) | `"hello world"` |
| dropdown | `{"ids": [1, 2]}` | `{"ids": [3]}` |
| people | `{"personsAndTeams": [{"id": N, "kind": "person"}]}` | `{"personsAndTeams": [{"id": 123, "kind": "person"}]}` |
| link | `{"url": "https://...", "text": "Label"}` | `{"url": "https://monday.com", "text": "Monday"}` |

For all 40+ column types, read `docs/column-types-reference.md` in this plugin directory.

**Step 4: Build the column_values JSON**

The `column_values` argument is a JSON **string** containing an object keyed by column ID:

```json
"{\"status_column_id\": {\"label\": \"Done\"}, \"date_column_id\": {\"date\": \"2026-01-15\"}}"
```

When using GraphQL variables, pass it as:

```javascript
const variables = {
  columnValues: JSON.stringify({
    status: { label: "Done" },
    date4: { date: "2026-01-15" }
  })
};
```

**Step 5: Build and test the mutation**

Use `change_multiple_column_values` for setting multiple columns at once (preferred over calling `change_simple_column_value` per column).

Follow the dry-run + confirm flow from Flow 2, Steps 4-5.

---

### Flow 4: Pagination

Pagination uses a two-phase cursor-based pattern. This is the most common source of bugs.

**Phase 1: Initial query with `items_page`**

`items_page` is nested inside `boards` or `groups`:

```graphql
query {
  boards(ids: [BOARD_ID]) {
    items_page(limit: 100, query_params: {
      rules: [{column_id: "status", compare_value: ["Done"]}]
    }) {
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

- `limit`: max 500 items per page (default 25)
- `query_params`: optional filtering (cannot combine with `cursor`)
- Save the returned `cursor` value
- If `cursor` is `null` after Phase 1, all items fit in one page — no Phase 2 needed

**Phase 2: Subsequent pages with `next_items_page`**

> **CRITICAL:** `next_items_page` MUST be a **root-level** query. Do NOT nest it inside `boards {}`. This is the single most common pagination bug.

```graphql
query {
  next_items_page(limit: 100, cursor: "YOUR_CURSOR_HERE") {
    cursor
    items {
      id
      name
      column_values { id text value }
    }
  }
}
```

- When `cursor` returns `null`, you have all items
- Keep the same field selection as Phase 1

**Full pagination loop in SDK code:**

Define queries in a `.graphql.ts` file and use code-generated types:

```typescript
// src/queries.graphql.ts
export const GET_BOARD_ITEMS_PAGE = `
  query GetBoardItemsPage($boardId: [ID!]!) {
    boards(ids: $boardId) {
      items_page(limit: 100) {
        cursor
        items { id name }
      }
    }
  }
`;

export const GET_NEXT_ITEMS_PAGE = `
  query GetNextItemsPage($cursor: String!) {
    next_items_page(limit: 100, cursor: $cursor) {
      cursor
      items { id name }
    }
  }
`;
```

```typescript
// src/index.ts
import 'dotenv/config';
import { ApiClient } from '@mondaydotcomorg/api';
import { GET_BOARD_ITEMS_PAGE, GET_NEXT_ITEMS_PAGE } from './queries.graphql';
import type { GetBoardItemsPageQuery, GetNextItemsPageQuery } from './generated/graphql';

const client = new ApiClient({ token: process.env.MONDAY_API_TOKEN! });

let allItems: GetBoardItemsPageQuery['boards'][0]['items_page']['items'] = [];

// Phase 1: Initial query
const firstPage = await client.request<GetBoardItemsPageQuery>(GET_BOARD_ITEMS_PAGE, { boardId: [BOARD_ID] });
allItems.push(...firstPage.boards[0].items_page.items);
let cursor = firstPage.boards[0].items_page.cursor;

// Phase 2: Subsequent pages (root-level!)
while (cursor) {
  const nextPage = await client.request<GetNextItemsPageQuery>(GET_NEXT_ITEMS_PAGE, { cursor });
  allItems.push(...nextPage.next_items_page.items);
  cursor = nextPage.next_items_page.cursor;
}
```

**Step: Live-test both phases**

Execute the Phase 1 query via `mcp__monday__all_monday_api`. If items exist and a cursor is returned, execute Phase 2 to verify the full loop works.

For the full pagination reference with filtering and edge cases, read `docs/pagination-patterns.md` in this plugin directory.

---

### Flow 5: Debug a Query

**Step 1: Get the failing query and error**

Ask the developer to paste:
- Their GraphQL query/mutation
- The error message or unexpected response

**Step 2: Run diagnostics**

Check these common issues in order:

| Check | How |
|-------|-----|
| Field names exist | `mcp__monday__get_type_details` to verify field names on the type |
| Using `items` instead of `items_page` | Look for `items {` directly on boards — should be `items_page` |
| `next_items_page` nested in boards | Must be at root level |
| Column value format wrong | `mcp__monday__get_column_type_info` for the column type |
| Missing required arguments | `mcp__monday__get_type_details` to check required args |
| `column_values` not stringified | Must be a JSON string, not a raw object |
| `query_params` + `cursor` combined | Cannot use both in the same `items_page` call |
| API version mismatch | Field may not exist in the pinned version |

**Step 3: Fix and re-test**

Apply the fix and execute the corrected query via `mcp__monday__all_monday_api`.

**Step 4: Present the corrected query**

Show the before/after diff and explain what was wrong.

---

## Error Handling

| Error | Action |
|-------|--------|
| "Field 'X' not found" | Introspect schema with `get_type_details` to find correct field name |
| "Argument 'X' required" | Check mutation signature for required args |
| `ColumnValueException` | Use `get_column_type_info` to get correct JSON format |
| `ComplexityException` | Reduce requested fields, add pagination limits, remove unnecessary nesting |
| "items" deprecation warning | Switch to `items_page` pattern (Flow 4) |
| MCP not connected | Tell user to check `/mcp` and ensure the monday server is listed |
| Rate limit / 429 | Suggest using `/monday-api:ops` for rate limit guidance |
| "How do I find column IDs?" | Use `mcp__monday__get_board_info` with the board ID — `columns { id title type }` |
