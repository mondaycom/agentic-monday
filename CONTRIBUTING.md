# Contributing

Guide for adding new plugins to the Monday Apps Framework marketplace.

## Plugin Manifest (plugin.json)

```json
{
  "name": "my-plugin",
  "description": "What the plugin does",
  "version": "0.1.0",
  "author": {
    "name": "your-name"
  },
  "repository": "https://github.com/mondaycom/agentic-monday-apps-framework",
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

### Skill Writing

- Include real, working code patterns (not theoretical examples)
- Reference actual monday.com API patterns and env var names
- Handle both local development and production scenarios
- Use MCP tool references (`mcp__monday-apps__*`) where applicable

## Testing Your Plugin

```bash
# Start claude with the local plugin loaded
claude --plugin-dir ./plugins/<your-plugin>
```

See that your skills / functionality are loaded

## Submitting

1. Fork the repository
2. Create a branch: `git checkout -b feature/my-plugin`
3. Add your plugin following the structure above
4. Update the marketplace manifest (`.claude-plugin/marketplace.json`)
5. Test locally
6. Submit a PR

## License

MIT
