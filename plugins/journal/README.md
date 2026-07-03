# Journal Plugin

A conversational journaling assistant for Claude Code. It interviews you about your day with genuine curiosity, then writes a polished Markdown journal entry to `documents/journal/YYYY-MM-DD.md`.

## Install

```
/plugin marketplace add <marketplace-url>
/plugin install journal@claude-plugins
```

## Usage

```
/journal:journal
```

Claude will ask you one open-ended question about your day, follow up on what's most interesting or emotionally resonant, and write the entry when the conversation feels complete (usually 4–8 exchanges).

## Output

Entries are saved to:

```
documents/journal/YYYY-MM-DD.md
```

Each entry includes a title, date, narrative prose written in first person, and a brief reflection section.
