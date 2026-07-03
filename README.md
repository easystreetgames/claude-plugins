# Claude Plugin Marketplace

A collection of Claude Code plugins — custom slash commands you can drop into any project.

## Installing a Plugin

```
/plugin marketplace add https://github.com/<your-username>/claude-plugins
/plugin install journal@claude-plugins
```

Then invoke the skill:

```
/journal:journal
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [journal](plugins/journal/) | Conversational journaling assistant — interviews you about your day and saves a Markdown entry |

## Contributing

Each plugin lives in `plugins/<name>/` with:
- `README.md` — what it does and how to use it
- `.claude-plugin/plugin.json` — plugin manifest (name, version, skills paths)
- `skills/<skill-name>/SKILL.md` — the skill prompt with frontmatter description
