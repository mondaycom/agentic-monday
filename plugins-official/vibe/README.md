# Vibe Plugin

Claude Code plugin for building monday.com apps with the [Vibe design system](https://vibe.monday.com) (`@vibe/core`).

## What Is Vibe?

Vibe is monday.com's open-source React component library and design system. When you build a monday.com app with a frontend, using Vibe ensures your UI looks and behaves like the rest of the platform — consistent styling, accessible components, and design tokens that respect the user's theme.

## Skills

### `/vibe-best-practices`

Verifies that your Vibe component usage follows design system standards and WCAG 2.1 AA accessibility requirements. Use it when:

- Writing new UI with `@vibe/core` components
- Reviewing code that imports from `@vibe/core`, `@vibe/core/next`, or `@vibe/icons`
- Fixing bugs in existing Vibe-based code

The skill uses the `@vibe/mcp` server to query live component metadata, accessibility requirements, and usage examples — so it always reflects the current Vibe API, not stale training data.

### `/vibe-upgrade`

Upgrades a project from one major version of Vibe to the next using the MCP migration tools. Supports:

- **v2 → v3** (`monday-ui-react-core` → `@vibe/core`)
- **v3 → v4** (`@vibe/core` → `@vibe/core` v4 with breaking changes)

## MCP Server

This plugin bundles the `@vibe/mcp` server. It provides these tools:

| Tool                               | Purpose                                                   |
| ---------------------------------- | --------------------------------------------------------- |
| `list-vibe-public-components`      | List all components in `@vibe/core` and `@vibe/core/next` |
| `get-vibe-component-metadata`      | Get props, types, and package info for a component        |
| `get-vibe-component-accessibility` | Get WCAG requirements for a component                     |
| `get-vibe-component-examples`      | Get React usage examples                                  |
| `list-vibe-icons`                  | Find available icon names from `@vibe/icons`              |
| `list-vibe-tokens`                 | Find design tokens (spacing, colors, border-radius)       |
| `v3-migration`                     | Get v2→v3 migration changes for a project                 |
| `v4-migration`                     | Get v3→v4 migration changes for a project                 |

The MCP server requires Node.js installed (runs via `npx @vibe/mcp`).

## Quick Start

Install Vibe in your monday.com app frontend:

```bash
npm install @vibe/core @vibe/icons
```

Add Vibe styles to your entry point:

```tsx
// src/main.tsx
import "@vibe/core/tokens";
```

Use components with the monday.com theme:

```tsx
import { Button, Text, Flex } from "@vibe/core";
import { Add } from "@vibe/icons";

function TaskActions({ onAdd }) {
  return (
    <Flex gap={8} align="center">
      <Text type="text1" weight="medium">
        No tasks yet
      </Text>
      <Button kind="primary" leftIcon={Add} onClick={onAdd}>
        Add task
      </Button>
    </Flex>
  );
}
```

## Resources

- [Vibe Documentation](https://vibe.monday.com)
- [Vibe on GitHub](https://github.com/mondaycom/vibe)
- [monday.com Developer Center](https://developer.monday.com)
- [@vibe/mcp on npm](https://www.npmjs.com/package/@vibe/mcp)
