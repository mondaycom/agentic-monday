# Contributing

Guide for adding new plugins to the Monday Apps Framework marketplace.

## Plugin Structure

Follow this structure (matches [agentic-builders-hub](https://github.com/dapulse/agentic-builders-hub) conventions):

```
plugins/my-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest (required)
├── skills/
│   └── my-skill/
│       └── SKILL.md         # Skill definition with YAML frontmatter (required)
├── agents/                  # Subagent definitions (optional)
│   └── agent-name.md
├── hooks/                   # Lifecycle hooks (optional)
│   └── hooks.json
├── scripts/                 # Supporting scripts (optional)
├── knowledge/               # Reference docs bundled with skill (optional)
└── README.md                # User-facing documentation (required)
```

## Plugin Manifest (plugin.json)

```json
{
  "name": "my-plugin",
  "description": "What the plugin does",
  "version": "0.1.0",
  "author": {
    "name": "your-name"
  },
  "repository": "https://github.com/gregr/agentic-monday-apps-framework",
  "keywords": ["monday", "monday-code", "your-keywords"]
}
```

## Skill Definition (SKILL.md)

Skills **must** include YAML frontmatter:

```markdown
---
name: my-skill
description: What this skill does
argument-hint: "[optional arguments]"
user-invocable: true
allowed-tools: ["Bash", "Write", "Read", "Glob", "Grep"]
---

# my-skill

Description of what the skill does.

## When to Use

- Trigger condition 1
- Trigger condition 2

## Instructions

Step-by-step instructions for Claude to execute...

## Notes

- Additional guidance
```

### Frontmatter Fields

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Skill name (used as slash command) |
| `description` | Yes | Human-readable description |
| `argument-hint` | No | Shows argument format hint |
| `user-invocable` | Yes | Set to `true` for slash commands |
| `allowed-tools` | No | Restrict which tools the skill can use |

## Guidelines

### monday-code Specifics

- Use `MNDY_MONGODB_CONNECTION_STRING` (not `MONGODB_URI`) for Document DB
- Use `@mondaycom/apps-sdk` version `^3.3.1` (not `"latest"`)
- Include `.mondaycoderc` for runtime selection
- Support both JWT token formats (production `dat.user_id` and dev `userId`)
- Always filter DB queries by `accountId` for multi-tenant isolation

### Skill Writing

- Include real, working code patterns (not theoretical examples)
- Reference actual monday.com API patterns and env var names
- Handle both local development and production scenarios
- Use MCP tool references (`mcp__monday-apps__*`) where applicable

## Testing Your Plugin

```bash
# Load the marketplace locally
/plugin marketplace add ./

# Install your plugin
/plugin install my-plugin@agentic-monday-apps-framework

# Test the skill
/my-skill
```

## Submitting

1. Fork the repository
2. Create a branch: `git checkout -b feature/my-plugin`
3. Add your plugin following the structure above
4. Update the marketplace manifest (`.claude-plugin/marketplace.json`)
5. Test locally
6. Submit a PR

## License

Contributions are licensed under MIT.
