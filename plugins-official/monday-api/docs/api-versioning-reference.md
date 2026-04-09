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

Given a version `YYYY-MM`, the schedule is:

| Phase | Timing |
|-------|--------|
| RC Start | ~1st of `YYYY-MM` |
| Current Start | ~15th of `YYYY-MM` |
| Maintenance Start | ~15th of the next quarter month |
| Deprecated | ~15th of the quarter after that |

For example, for version `YYYY-04`: RC ~Apr 1, Current ~Apr 15, Maintenance ~Jul 15, Deprecated ~Oct 15.

At any point in time there is one **Release Candidate** (`current quarter`), one **Current** version (`previous quarter`), and one **Maintenance** version (`two quarters ago`). Check the [official changelog](https://developer.monday.com/api-reference/docs/api-versioning) for the currently active versions.

## Version Header Behavior

| What you send | What you get |
|--------------|--------------|
| Valid current version (e.g., `YYYY-01`) | That version |
| Valid maintenance version (e.g., `YYYY-10`) | That version |
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

## Notable Breaking Changes

For version-specific breaking changes, always refer to the [official API changelog](https://developer.monday.com/api-reference/docs/api-versioning) — it is the authoritative and up-to-date source.

Common historical patterns to watch for when migrating between versions:

- **`items` → `items_page`**: The `items` field on boards was deprecated in favor of `items_page` with cursor-based pagination. This is the single largest breaking change most developers encounter.
- **Column value types**: Type-specific fragments (`... on StatusValue`, `... on DateValue`) replaced the generic `value` JSON string approach.
- **ID types**: Several fields changed from `Int` to `ID` (string) type.
- **SDK constructor changes**: `query()` was replaced by `request()` with a `{ token }` constructor in older SDK versions.

## Migration Checklist

When upgrading between versions:

1. Check the [official changelog](https://developer.monday.com/api-reference/docs/api-versioning) for breaking changes
2. Search your codebase for deprecated fields (use `/monday-api:migrate` skill)
3. Test queries against the new version
4. Update the version pin in your code
5. Monitor for issues after deployment

## SDK Version ↔ API Version Mapping

Each major SDK version pins a specific default API version. Upgrading the SDK major version may change your effective API version if you're not pinning explicitly.

Check the [SDK changelog](https://github.com/mondaycom/monday-sdk-js/blob/master/CHANGELOG.md) to see which API version a given SDK release defaults to.
