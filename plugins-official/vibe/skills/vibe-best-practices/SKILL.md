---
name: vibe-best-practices
description: Ensures Vibe design system (@vibe/core) components follow best practices and accessibility standards. Use when writing or reviewing UI code in monday.com apps that import from @vibe/core or @vibe/core/next.
user-invocable: true
allowed-tools:
  [
    Read,
    Glob,
    Grep,
    mcp__vibe__get-vibe-component-metadata,
    mcp__vibe__get-vibe-component-accessibility,
    mcp__vibe__list-vibe-public-components,
    mcp__vibe__list-vibe-icons,
    mcp__vibe__list-vibe-tokens,
    mcp__vibe__get-vibe-component-examples,
  ]
---

# Vibe Best Practices

## Overview

Verify Vibe component usage follows design system standards and WCAG 2.1 AA accessibility requirements using MCP tools before implementation or during code review.

**Core Principle:** Check compliance ALWAYS — for quick fixes, working code, urgent tasks, and when matching existing patterns. No exceptions.

## When to Use

**Always use this skill when:**

- Writing new UI with `@vibe/core` or `@vibe/core/next` components
- Reviewing code that imports from `@vibe/core`, `@vibe/core/next`, or `@vibe/icons`
- Fixing bugs in existing Vibe components
- User mentions Vibe, design system, accessibility, or WCAG concerns
- Building monday.com apps with a React frontend

## When NOT to Use

- Pure backend/API code with no UI components
- Projects that don't use the Vibe design system
- Non-React projects (Vibe is React-specific)

---

## ⚠️ CRITICAL: Scan Imports First

**BEFORE analyzing any code, check imports (lines 1–10):**

1. Are components from `@vibe/core/next` when available? (Call `get-vibe-component-metadata` to check)

Import violations are the most commonly missed. **Stop and check imports first.**

---

## Quick Reference

### Decision Matrix

| Code Pattern                            | MCP Tool to Call              | Why                              |
| --------------------------------------- | ----------------------------- | -------------------------------- |
| `import { Dropdown } from "@vibe/core"` | `get-vibe-component-metadata` | Check if /next version exists    |
| `<Button kind={Button.kinds.PRIMARY}>`  | `get-vibe-component-metadata` | Get valid string prop values     |
| `<div onClick={handler}>`               | `list-vibe-public-components` | Find accessible Vibe alternative |
| `style={{ fontSize: '16px' }}`          | `list-vibe-tokens`            | Replace with design tokens       |
| `monday-ui-react-core` imports          | `list-vibe-public-components` | Find v3 equivalents              |

### MCP Tools

| Tool                               | When to Use                                            |
| ---------------------------------- | ------------------------------------------------------ |
| `list-vibe-public-components`      | Check if a Vibe component exists before using raw HTML |
| `get-vibe-component-metadata`      | Verify component API, props, and available versions    |
| `get-vibe-component-accessibility` | Get accessibility requirements for a component         |
| `get-vibe-component-examples`      | See correct usage patterns                             |
| `list-vibe-icons`                  | Find available icon names from `@vibe/icons`           |
| `list-vibe-tokens`                 | Find design tokens (spacing, colors, border-radius)    |

### Compliance Rules Summary

| #   | Rule                                 | Quick Check                                 |
| --- | ------------------------------------ | ------------------------------------------- |
| 1   | Use `@vibe/core/next` when available | Call `get-vibe-component-metadata` to check |
| 2   | Use `Text`/`Heading` for typography  | Never manual text styling                   |
| 3   | String literals for props            | `"primary"` not `Button.kinds.PRIMARY`      |
| 4   | No mixing v2 and v3                  | Never `monday-ui-react-core`                |
| 5   | No `!important` in CSS               | Breaks Vibe theme consistency               |
| 6   | No inline styles on Vibe components  | No `style={}` prop                          |
| 7   | Use design tokens                    | CSS custom properties, not hardcoded values |

---

## Mandatory Workflow

**STOP: Read this before writing or reviewing any Vibe code.**

### Before Implementation

1. **Scan imports first** — check every import against Rule 1

2. **Check component availability:**

   ```
   list-vibe-public-components
   ```

3. **Verify component API:**

   ```
   get-vibe-component-metadata ComponentName
   ```

4. **Check accessibility requirements:**

   ```
   get-vibe-component-accessibility ComponentName
   ```

5. **See usage examples:**
   ```
   get-vibe-component-examples ComponentName
   ```

### During Code Review

**MANDATORY — no exceptions:**

1. **Scan imports** for Vibe packages (lines 1–10 typically)
2. **Call MCP tools** for EACH component found
3. **Verify compliance** with all 7 rules
4. **Check accessibility** against WCAG 2.1 AA
5. **Report violations** using the standard format below

**Critical: Do NOT skip MCP tools to save time. Tool calls are fast and prevent mistakes.**

---

## Compliance Rules

### Rule 1: Use `@vibe/core/next` When Available

1. Call `get-vibe-component-metadata` for each component
2. Check if a `/next` version exists in the response
3. If `/next` version available, import from `@vibe/core/next`
4. If no `/next` version, import from `@vibe/core`

```tsx
// Violation:
import { Dropdown } from "@vibe/core";

// Compliant (after verifying /next exists via MCP):
import { Dropdown } from "@vibe/core/next";
```

### Rule 2: Use `Text` and `Heading` for Typography

1. Call `get-vibe-component-metadata` for `Text` and `Heading`
2. Call `get-vibe-component-examples` for proper usage patterns
3. Replace manual text styling with appropriate Vibe components

```tsx
// Violation:
<p style={{ fontSize: "16px", fontWeight: "bold" }}>Task name</p>

// Compliant:
<Text type="text1" weight="bold">Task name</Text>
```

### Rule 3: String Literals for Props

1. Call `get-vibe-component-metadata` for the component
2. Check the metadata for valid string values (e.g., "primary", "secondary")
3. Use string literals from metadata — never static class properties

```tsx
// Violation:
<Button kind={Button.kinds.PRIMARY}>Submit</Button>

// Compliant:
<Button kind="primary">Submit</Button>
```

### Rule 4: No Mixing v2 and v3

1. Scan all imports in the file
2. Flag any imports from `monday-ui-react-core` (v2)
3. Call `list-vibe-public-components` to find v3 equivalents
4. All Vibe components must come from `@vibe/core` or `@vibe/core/next`

```tsx
// Violation:
import { Button } from "monday-ui-react-core";

// Compliant:
import { Button } from "@vibe/core";
```

### Rule 5: No `!important` in CSS

1. Scan CSS files for `!important` declarations
2. Call `get-vibe-component-metadata` to find the proper prop or token
3. Use component props or design tokens instead of CSS overrides

```css
/* Violation: */
.myButton {
  background-color: blue !important;
}

/* Compliant — use component props or tokens: */
.myButton {
  background-color: var(--primary-color);
}
```

### Rule 6: No Inline Styles on Vibe Components

1. Scan Vibe components for `style={{}}` props
2. Call `get-vibe-component-metadata` to find proper props (e.g., `gap`, `margin`)
3. Use component props or CSS classes instead

```tsx
// Violation:
<Flex style={{ gap: "16px" }}>...</Flex>

// Compliant:
<Flex gap="medium">...</Flex>
// (verify prop names via get-vibe-component-metadata)
```

### Rule 7: Use Design Tokens

1. Call `list-vibe-tokens` to find available design tokens
2. Replace hardcoded values with CSS custom properties

```css
/* Violation: */
.container {
  margin-bottom: 16px;
  color: #323338;
}

/* Compliant: */
.container {
  margin-bottom: var(--space-16);
  color: var(--primary-text-color);
}
```

---

## MCP Tools Reference

### `list-vibe-public-components`

Lists all public components from `@vibe/core` and `@vibe/core/next`.

**When to use:** Before implementing — check if Vibe has a component before reaching for raw HTML.

```tsx
// DON'T immediately write:
<div onClick={handleClick}>Click me</div>

// DO check first:
// list-vibe-public-components → shows Clickable exists
<Clickable onClick={handleClick}>Click me</Clickable>
```

---

### `get-vibe-component-metadata`

Returns props, types, package (`@vibe/core` vs `/next`), deprecation status, and related components.

**When to use:** ALWAYS before using a component — verify API and check for `/next` version.

```tsx
// Call: get-vibe-component-metadata Button
// Response: kind accepts "primary", "secondary", "tertiary"

// Violation:
<Button kind={Button.kinds.PRIMARY}>

// Compliant:
<Button kind="primary">
```

---

### `get-vibe-component-accessibility`

Returns WCAG requirements, required props (e.g., `aria-label`), keyboard behavior, and screen reader announcements.

**When to use:** MANDATORY for EVERY Vibe component — before implementation and during review.

```tsx
// Call: get-vibe-component-accessibility IconButton
// Response: aria-label required for icon-only buttons

// Violation:
<IconButton icon={Delete} onClick={handleDelete} />

// Compliant:
<IconButton icon={Delete} aria-label="Delete item" onClick={handleDelete} />
```

---

### `get-vibe-component-examples`

Returns React usage examples and patterns.

**When to use:** Learning a new component's API or understanding correct usage patterns.

```tsx
// Call: get-vibe-component-examples Modal
// Shows: Modal with ModalHeader, ModalContent, ModalFooter structure
```

---

### `list-vibe-icons`

Lists available icons with descriptions, categories, and correct names.

**When to use:** Finding the right icon name before importing from `@vibe/icons`.

```tsx
// Call: list-vibe-icons close
// Response: Close, CloseSmall, CloseCircle

import { Close } from "@vibe/icons";
<IconButton icon={Close} aria-label="Close dialog" />;
```

---

### `list-vibe-tokens`

Lists design tokens (spacing, colors, border-radius, typography).

**When to use:** Replacing hardcoded CSS values with Vibe tokens.

```css
/* Call: list-vibe-tokens spacing */
/* Response: --space-16: 16px */

.container {
  margin-bottom: var(--space-16);
}
```

---

## Accessibility (WCAG 2.1 AA)

### Always Call `get-vibe-component-accessibility`

For EVERY Vibe component, call `get-vibe-component-accessibility` to get specific requirements. Don't rely on assumptions — requirements vary by component and can change across versions.

### Prefer Vibe Components Over Raw HTML

```tsx
// Raw HTML (avoid):
<div onClick={handleClick} role="button" tabIndex={0} onKeyDown={...}>

// Vibe component (preferred — handles accessibility automatically):
<Clickable onClick={handleClick}>
```

### Common Issues the MCP Tool Will Catch

- Missing `aria-label` on icon-only buttons
- Missing `alt` text on images
- Missing labels on form inputs (`title` or `inputAriaLabel` on `TextField`)
- Missing keyboard accessibility on interactive elements

### Before Flagging — Investigate First

Check for:

- **Props spreading:** `{...props}` may pass accessibility props
- **External labels:** `<label htmlFor>` elsewhere in the file
- **Wrapper components:** `<Tooltip>` may provide an accessible name
- **Component composition:** `<ListItem>` rendering `<li>` is correct if `<List>` provides `<ul>`

Only flag confirmed violations, or use "Potential Issue" if uncertain.

---

## Violation Comment Formats

### Vibe Standards Violation

```
⚠️ Vibe Standards Violation - Rule {number}

{Brief description}

**Rule:** {rule-name}
**Affected code:** Lines {start}-{end} in `{filename}`

**Fix:** {Specific remediation with code example}
```

**Example:**

```
⚠️ Vibe Standards Violation - Rule 3

Button prop uses static property instead of string literal.

**Rule:** String literals for props
**Affected code:** Line 12 in `TaskCard.tsx`

**Fix:**
- <Button kind={Button.kinds.PRIMARY}>Submit</Button>
+ <Button kind="primary">Submit</Button>
```

### Accessibility Violation

```
⚠️ Accessibility Violation - WCAG {criterion}

{Brief description}

**Element:** `{ComponentName}`
**WCAG:** {criterion-number} - {criterion-name}
**Affected code:** Lines {start}-{end} in `{filename}`

**Fix:** {Suggest Vibe component first, then manual fix}
```

**Example:**

```
⚠️ Accessibility Violation - WCAG 4.1.2

Icon-only button missing accessible name.

**Element:** `<IconButton>`
**WCAG:** 4.1.2 - Name, Role, Value
**Affected code:** Line 24 in `TaskCard.tsx`

**Fix:** Add aria-label prop:
<IconButton icon={Delete} aria-label="Delete task" onClick={handleDelete} />
```

---

## Anti-Rationalization Rules

**Never skip compliance checks. No exceptions.**

### Common Rationalizations to Reject

**"This is a quick fix, I'll skip the checks"**
Even one-line changes can introduce import violations (Rule 1). Scanning imports takes 10 seconds.

**"The code works, no need to change"**
Working code can import from `@vibe/core` instead of `/next`, use static properties like `Button.kinds.PRIMARY`, or miss `aria-label` — all while functioning perfectly. Functionality ≠ Compliance.

**"I'll match the existing pattern for consistency"**
Propagating anti-patterns creates more debt. Implement correctly per Vibe standards, then flag that the existing code also violates them.

**"There are too many issues to report all of them"**
Report every violation. Group by type if needed, prioritize: Critical (accessibility) > High (import violations) > Medium (static props).

**"I'll skip MCP tools to save time"**
MCP tools are fast (<500ms). They are the source of truth for Vibe APIs. Without them, you're guessing at prop values and accessibility requirements.

**"That's out of scope for this task"**
Vibe compliance is always in scope when touching files with Vibe imports. Flag violations even if fixing them is out of scope — let the user decide.

### Handling Pressure Scenarios

**Urgent bug fix:**

1. Fix the bug ✓
2. While the file is open, scan imports (5 seconds)
3. Report: "Fixed bug. Also found 2 Vibe violations to address separately."

**"Match existing code used in 50+ files":**

1. Implement new code following Vibe standards ✓
2. Flag that the existing pattern violates Rule X
3. Suggest: "New code is correct. Consider migrating the existing 50+ files."

**"Been in production 6 months, works fine":**

1. Acknowledge it works ✓
2. Report violations anyway
3. Frame as: "Works well but violates Rule X. Consider fixing in the next update."

---

## Final Checklist

**Component Compliance:**

- [ ] All components from `@vibe/core/next` when available
- [ ] All typography uses `Text`/`Heading`
- [ ] No static properties (no `.kinds.`, `.sizes.`, `.types.`)
- [ ] No `monday-ui-react-core` imports
- [ ] No `!important` in CSS
- [ ] No `style={{}}` props on Vibe components
- [ ] All hardcoded CSS values replaced with design tokens

**Accessibility:**

- [ ] Called `get-vibe-component-accessibility` for each component
- [ ] Icon-only buttons have `aria-label`
- [ ] Images have `alt` attributes
- [ ] Form inputs have labels (`title` or `inputAriaLabel`)
- [ ] Interactive elements are keyboard accessible
- [ ] Vibe components used instead of raw interactive HTML

**MCP Tools Used:**

- [ ] `list-vibe-public-components` — checked component availability
- [ ] `get-vibe-component-metadata` — verified props and package
- [ ] `get-vibe-component-accessibility` — checked requirements for each component
- [ ] `get-vibe-component-examples` — reviewed usage patterns
