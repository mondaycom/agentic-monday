---
name: vibe-upgrade
description: Upgrades a project to a new major version of Vibe (@vibe/core) using MCP migration tools. Use when upgrading from v2 to v3 (monday-ui-react-core → @vibe/core) or from v3 to v4.
argument-hint: "[project-path]"
user-invocable: true
allowed-tools: [Read, Glob, Grep, Bash, Edit, mcp__vibe__v3-migration, mcp__vibe__v4-migration]
---

# Vibe Upgrade

## CRITICAL: Always Use the MCP Migration Tool

Do NOT attempt to:
- Guess breaking changes from training knowledge
- Fetch changelogs manually and apply them yourself
- Use grep to discover what needs migrating

The Vibe MCP migration tools have authoritative, version-specific knowledge of every breaking change. They are the only reliable way to perform this migration correctly.

---

## Workflow

### Step 1 — Determine Project Path and Current Version

If `$ARGUMENTS` is provided, use it as the project path. Otherwise use the current working directory. In a monorepo, use the path to the specific workspace package that uses `@vibe/core`, not the repo root.

Read `<project-path>/package.json` to find the installed version:

```json
"@vibe/core": "^3.12.0"
```

Or for v2:

```json
"monday-ui-react-core": "^2.x.x"
```

Select the MCP tool based on current major version:

| Current | Target | MCP Tool |
|---------|--------|----------|
| v2.x (`monday-ui-react-core`) | v3 (`@vibe/core`) | `mcp__vibe__v3-migration` |
| v3.x (`@vibe/core`) | v4 | `mcp__vibe__v4-migration` |

**If neither `monday-ui-react-core` nor `@vibe/core` is in `package.json`:** Stop and inform the user this skill does not apply.

**If the version is outside the table above (e.g., v1.x or already v4):** Stop and inform the user that no MCP migration tool exists for that version path.

**If migrating v2 → v4 (two hops):** Complete the v2→v3 migration in full (all 5 steps), then restart from Step 1 for the v3→v4 migration.

---

### Step 2 — Run the MCP Migration Tool

Call the appropriate tool with the **absolute** project path:

```
# For v3 → v4:
mcp__vibe__v4-migration({ projectPath: "/absolute/path/to/project" })

# For v2 → v3:
mcp__vibe__v3-migration({ projectPath: "/absolute/path/to/project" })
```

The tool returns:
- A list of files that need changes with before/after replacements for each
- Commands to run (including the version bump install command)

**Wait for the tool to complete before touching any files.**

If the tool fails or returns an error: stop, report the error to the user, and do not attempt a manual migration.

---

### Step 3 — Apply Migration Changes

For each file in the tool output:

1. Read the current file with the Read tool
2. Apply the before → after replacements exactly as specified
3. Write the updated file with the Edit tool

**Rules:**
- Apply ALL changes from the tool output — do not skip files
- Apply changes exactly as specified — do not improvise
- If a replacement pattern is not found in the file (already applied or file drifted), skip that replacement and note it — do not guess an alternative

---

### Step 4 — Install Updated Dependencies

Run the install command provided by the MCP tool output. It will include the correct package version.

Detect the project's package manager from lock files first:

| Lock File | Package Manager |
|-----------|----------------|
| `yarn.lock` | `yarn` |
| `pnpm-lock.yaml` | `pnpm` |
| `package-lock.json` | `npm` |

```bash
# Example (use the exact command from the tool output):
npm install @vibe/core@4
```

---

### Step 5 — Validate

Run build and tests:

```bash
# Yarn:
yarn build && yarn test

# npm:
npm run build && npm test
```

Use the Grep tool to confirm any deprecated patterns flagged by the MCP tool are gone. Report any remaining issues to the user.

---

## Common Mistakes

| Mistake | Correct Behavior |
|---------|-----------------|
| Skipping the MCP tool and using training knowledge | Always run the MCP tool first |
| Only applying changes to some files | Apply ALL changes from the tool output |
| Calling the MCP tool with a relative path | Always use an absolute path |
| Running v2→v4 in one hop | Complete v2→v3 fully, then run v3→v4 |
| Continuing after a tool error | Stop and report; never attempt manual migration |
| Using wrong package manager | Detect from lock files before running install |

---

## Done Criteria

- [ ] MCP migration tool was run and all its output was applied
- [ ] Dependencies installed at the new major version
- [ ] No TypeScript errors in files that import from `@vibe/core`
- [ ] Build passes
- [ ] Tests pass (or remaining failures documented and handed off to the developer)

---

## After the Upgrade

Once the migration is complete, use `/vibe-best-practices` to verify that all components in the upgraded project follow the current Vibe standards — particularly:

- Checking for components that now have a `/next` version available
- Confirming icon imports use `@vibe/icons`
- Verifying string literal props (not static class properties)
