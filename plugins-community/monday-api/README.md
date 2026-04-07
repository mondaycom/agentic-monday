# monday-api

Simplifies the monday.com public GraphQL API for external developers. Four skills covering the full API lifecycle: project setup, query building with live testing, version migration, and production operations.

## Installation

```bash
/install-plugin monday-api
```

Verify the MCP server is connected:

```
/mcp
```

## Prerequisites

- The **monday** MCP server must be connected (provides schema introspection and query execution)
- A monday.com API token for live query testing

## Skills

### `/monday-api:setup`

Bootstrap a project with the `@mondaydotcomorg/api` SDK. Installs dependencies, configures authentication, pins the API version, and optionally sets up TypeScript codegen.

### `/monday-api:build-query`

Build, test, and debug GraphQL queries and mutations. Introspects the live schema, formats column values correctly, handles pagination, and validates every query by executing it against the real API.

### `/monday-api:migrate`

Migrate between quarterly API versions. Detects your current version, identifies breaking changes, scans your codebase for affected patterns, guides code updates, and tests migrated queries.

### `/monday-api:ops`

Production operations guidance. Covers error handling (HTTP 200 gotcha), rate limits (4 independent axes), complexity budgeting, retry strategies, webhooks vs polling, caching, and a production readiness checklist.

## How It Works

```
Developer request
       |
       v
  Claude Code + monday-api plugin
       |
       +-- get_graphql_schema  (introspect types)
       +-- get_type_details    (inspect fields)
       +-- get_column_type_info (column JSON formats)
       +-- all_monday_api      (execute & validate)
       |
       v
  Working, tested query + SDK code
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| MCP not connected | Run `/mcp` and check the monday server is listed |
| Auth errors | Verify your API token is valid and has the right permissions |
| Query complexity exceeded | Reduce requested fields, add pagination limits |
| Column value parse error | Use `/monday-api:build-query` to get the exact JSON format |
