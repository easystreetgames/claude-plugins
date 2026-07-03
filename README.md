# Claude Plugin Marketplace

A collection of Claude Code plugins — custom slash commands you can drop into any project.

## Installing a Plugin

Copy the desired plugin's command file into your project's `.claude/commands/` directory:

```
.claude/
  commands/
    journal.md   ← paste here
```

Then use it in Claude Code with `/<plugin-name>`.

## Plugins

| Plugin | Description |
|--------|-------------|
| [journal](plugins/journal/) | Conversational journaling assistant — interviews you about your day and saves a Markdown entry |

## Contributing

Each plugin lives in `plugins/<name>/` with:
- `README.md` — what it does and how to use it
- `command.md` — the file to copy into `.claude/commands/`
- `plugin.json` — metadata (name, version, author, description)
