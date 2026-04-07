---
name: migrate
description: Migrate between monday.com API versions. Identifies your current version, shows breaking changes, guides code updates, and tests migrated queries. Use when a new API version is released or you need to upgrade from a deprecated version.
argument-hint: "<current-version> [target-version]"
user-invocable: true
allowed-tools: [Bash, Read, Write, Edit, Glob, Grep, AskUserQuestion, mcp__monday__get_graphql_schema, mcp__monday__get_type_details, mcp__monday__all_monday_api]
---

# API Version Migration

Guide developers through migrating between quarterly monday.com API versions. Detects current version, identifies breaking changes, scans the codebase, and tests migrated queries.

## Scope

- Helps external developers upgrade between API versions
- Covers schema changes, deprecated fields, and code migration
- Does NOT cover SDK package upgrades (major npm version bumps) — that's a separate concern

---

## API Version Lifecycle

| Phase | Duration | Behavior |
|-------|----------|----------|
| Release Candidate | ~2 weeks before quarter start | Available for testing, not default, may change |
| Current | 1 quarter (3 months) | Default version, fully supported, bug fixes only |
| Maintenance | 1 quarter | Still works, no new features, migration recommended |
| Deprecated | After maintenance | May stop working, migration required |

**Version format:** `YYYY-MM` where MM is `01`, `04`, `07`, `10`

**Current stable (as of April 2026):** `2026-01`

**Important:** If you send an invalid or deprecated version in the `API-Version` header, the API silently falls back to the Current version — no error is returned. This can mask version-related bugs.

For version history and known breaking changes, read `docs/api-versioning-reference.md` in this plugin directory.

---

## Workflow

### Step 1: Detect Current Version

Search the codebase for version references using these bash commands:

```bash
# Find SDK version pins
grep -rn "apiVersion" . --include="*.ts" --include="*.js" --include="*.json"

# Find raw HTTP version headers
grep -rn "API-Version\|api-version" . --include="*.ts" --include="*.js"

# Find hard-coded version strings (all quarterly versions)
grep -rn "202[0-9]-0[1-9]\|202[0-9]-10" . --include="*.ts" --include="*.js" --include="*.env*"

# Find setApiVersion calls (older SDK pattern)
grep -rn "setApiVersion" . --include="*.ts" --include="*.js"
```

Check results in:
- SDK initialization code (`new ApiClient({ apiVersion: ... })`)
- HTTP headers (`'API-Version': '...'`)
- Environment variables and `.env` files
- Configuration files

If no version is found, the project is using the default (Current) version — this is a risk because the default changes quarterly.

### Step 2: Determine Target Version

- **Default:** latest Current version (`2026-01` as of now)
- If the user specifies a target, validate it's a real version (format `YYYY-MM`, month in `01|04|07|10`)
- Show the version gap:

```
Current version: 2025-04
Target version:  2026-01
Gap: 3 versions (2025-04 → 2025-07 → 2025-10 → 2026-01)
```

### Step 3: Identify Breaking Changes

**Approach 1: Schema inspection (preferred)**

Use `mcp__monday__get_graphql_schema` with `operationType: "read"` to get all available query fields. Then use `operationType: "write"` for mutations. Look for:
- Fields or mutations that exist in your current code but are missing from the schema response (removed)
- Argument names that differ from what your code uses (renamed)

Then use `mcp__monday__get_type_details` for types your code depends on (e.g., `Item`, `Board`, `ColumnValue`). Compare the fields listed against what your code expects.

Example — if your code reads `item.column_values[0].value` but `get_type_details` shows the field is now called `display_value`, that's a breaking change.

**Approach 2: Known breaking changes reference**

Reference `docs/api-versioning-reference.md` for documented changes per version.

**Common breaking changes between versions:**
- Fields removed or renamed
- Argument types changed (e.g., `Int` → `ID`)
- Response shapes modified (especially `column_values`)
- Pagination changes (`items` → `items_page` was a major one)
- New required arguments added
- Enum values changed

### Step 4: Scan Codebase for Affected Patterns

Search the developer's code for patterns that may break. Run these Bash commands:

```bash
# Check for old pagination pattern
grep -rn "boards.*{[^}]*items {" src/ --include="*.ts" --include="*.js"

# Check for hard-coded version strings
grep -rn "202[0-9]-[0-9][0-9]" src/ --include="*.ts" --include="*.js"

# Check for deprecated fields commonly removed between versions
grep -rn "column_values.*\.value\b" src/ --include="*.ts" --include="*.js"

# Check for old items query (IDs-based, not items_page)
grep -rn "items(ids:" src/ --include="*.ts" --include="*.js"
```

Adjust paths as needed (`src/` may be different in the developer's project).

Present findings as an impact table:

```markdown
## Migration Impact: 2025-10 → 2026-01

| File | Line | Issue | Required Action |
|------|------|-------|-----------------|
| src/api.ts | 42 | Uses deprecated `items` field | Change to `items_page` with cursor pagination |
| src/board.ts | 18 | Old `column_values` format | Update to type-specific fragments |
| src/sync.ts | 95 | Uses removed `complexity_level` field | Remove field from query |
| lib/config.ts | 7 | Version pinned to `2025-10` | Update to `2026-01` |
```

If no affected patterns are found, tell the developer their code is likely compatible.

### Step 5: Guide Code Changes

For each affected location from Step 4:

1. Show the **before** code (current)
2. Show the **after** code (migrated)
3. Explain **why** the change is needed

Example:
```
### src/api.ts:42 — Switch from items to items_page

BEFORE:
boards(ids: [123]) { items { id name } }

AFTER:
boards(ids: [123]) { items_page(limit: 100) { cursor items { id name } } }

WHY: The `items` field on boards is deprecated. Use `items_page` for
cursor-based pagination with filtering support.
```

Offer to apply the changes using Edit tool.

### Step 6: Test Migrated Queries

For each modified query/mutation:

1. Execute via `mcp__monday__all_monday_api` to verify it works
2. Check the response shape matches expectations
3. Confirm no errors in the `errors` array

If a query fails, diagnose and fix before moving to the next one.

### Step 7: Update Version Pin

After all queries pass:

1. Update `apiVersion` in SDK initialization
2. Update `API-Version` headers in raw HTTP calls
3. Update any environment variables or config files

Show the final change:
```typescript
// BEFORE
const client = new ApiClient({ token: TOKEN, apiVersion: '2025-10' });

// AFTER
const client = new ApiClient({ token: TOKEN, apiVersion: '2026-01' });
```

---

## Post-Migration Recommendations

- **Test against the next RC** — when the next Release Candidate is available, test early
- **Set a calendar reminder** — API versions rotate quarterly, plan migrations ahead
- **Monitor for deprecation warnings** — the API may return warnings in the response `extensions`
- **Update SDK package** — new API versions often correspond to new SDK major versions with matching defaults
- **Gradual migration with versionOverride** — if you need to migrate incrementally, the SDK supports per-request version overrides: `client.request(query, variables, { versionOverride: '2026-01' })`. This lets you migrate one query at a time before switching the entire client.

---

## Error Handling

| Error | Action |
|-------|--------|
| Can't detect current version | Ask the developer, check `package.json` for SDK version hints |
| Unknown target version | List available versions with their status (Current, Maintenance, Deprecated) |
| Breaking change not documented | Check the [official changelog](https://developer.monday.com/api-reference/docs/api-versioning), then test the specific query via `mcp__monday__all_monday_api` with both old and new version syntax to see which one succeeds |
| Query fails after migration | Check if fields were renamed (not removed) — use `get_type_details` to find the new name |
| Version header silently ignored | Remind: invalid versions fall back to Current silently — verify the version string format |
