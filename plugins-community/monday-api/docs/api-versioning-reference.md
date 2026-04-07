# API Versioning Reference

Reference for monday.com API version lifecycle, schedule, and known breaking changes.

## Version Lifecycle

```
    RC (testing)          Current (stable)       Maintenance (legacy)     Deprecated
  ┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌──────────┐
  │ ~2 weeks     │ ──> │ 3 months         │ ──> │ 3 months         │ ──> │ Retired  │
  │ May change   │     │ Bug fixes only   │     │ No new features  │     │ May fail │
  │ Not default  │     │ Default version  │     │ Still works      │     │          │
  └──────────────┘     └──────────────────┘     └──────────────────┘     └──────────┘
```

## Version Schedule

Versions follow a quarterly cadence with format `YYYY-MM` where MM is `01`, `04`, `07`, `10`.

| Version | RC Start | Current Start | Maintenance Start | Deprecated |
|---------|----------|---------------|-------------------|------------|
| `2026-04` | ~Apr 1, 2026 | Apr 15, 2026 | Jul 15, 2026 | Oct 15, 2026 |
| `2026-01` | ~Jan 1, 2026 | Jan 15, 2026 | Apr 15, 2026 | Jul 15, 2026 |
| `2025-10` | ~Oct 1, 2025 | Oct 15, 2025 | Jan 15, 2026 | Apr 15, 2026 |
| `2025-07` | ~Jul 1, 2025 | Jul 15, 2025 | Oct 15, 2025 | Jan 15, 2026 |
| `2025-04` | ~Apr 1, 2025 | Apr 15, 2025 | Jul 15, 2025 | Oct 15, 2025 |
| `2025-01` | ~Jan 1, 2025 | Jan 15, 2025 | Apr 15, 2025 | Jul 15, 2025 |

**As of April 2026:**
- **Release Candidate:** `2026-04`
- **Current (default):** `2026-01`
- **Maintenance:** `2025-10`

## Version Header Behavior

| What you send | What you get |
|--------------|--------------|
| Valid current version (e.g., `2026-01`) | That version |
| Valid maintenance version (e.g., `2025-10`) | That version |
| No `API-Version` header | Current version (changes quarterly!) |
| Invalid version string | Current version (silent fallback, no error) |
| Deprecated version | Maintenance version (silent fallback) |

**Always pin your version explicitly.** Silent fallbacks mask bugs.

## Special Version Values

The API also accepts these aliases instead of `YYYY-MM`:

| Alias | Resolves to |
|-------|-------------|
| `dev` | Latest development version (unstable, for testing only) |
| `deprecated` | Oldest still-supported version |
| `maintenance` | Current maintenance version |
| `current` | Current stable version |
| `release_candidate` | Latest RC version |

These are useful for testing but should NOT be used in production — they resolve to different actual versions over time.

## Notable Breaking Changes by Version

### 2026-01
- Default API version for `@mondaydotcomorg/api` v13+
- Timeout/cancellation via `timeoutMs` option added to SDK

### 2025-10
- Default API version for `@mondaydotcomorg/api` v12
- Various field additions and type refinements

### 2025-07
- Per-request `versionOverride` option added to SDK
- `SeamlessApiClientError` now includes partial data (SDK v11.1.0)

### Historical Major Changes
- **`items` → `items_page`**: The `items` field directly on boards was deprecated in favor of `items_page` with cursor-based pagination. This is the single largest breaking change most developers encounter.
- **Column value types**: Type-specific fragments (`... on StatusValue`, `... on DateValue`) replaced the generic `value` JSON string approach in newer versions.
- **ID types**: Several fields changed from `Int` to `ID` (string) type across versions.
- **Constructor changes**: SDK v6 changed from `query()` to `request()` method, with `{ token }` constructor.

## Migration Checklist

When upgrading between versions:

1. Check the [official changelog](https://developer.monday.com/api-reference/docs/api-versioning) for breaking changes
2. Search your codebase for deprecated fields (use `/monday-api:migrate` skill)
3. Test queries against the new version
4. Update the version pin in your code
5. Monitor for issues after deployment

## SDK Version ↔ API Version Mapping

Each major SDK version updates the default API version:

| SDK Version | Default API Version |
|-------------|-------------------|
| v14.x | `2026-01` |
| v13.x | `2026-01` |
| v12.x | `2025-10` |
| v11.x | `2025-07` |

Upgrading the SDK major version may change your effective API version if you're not pinning explicitly.
