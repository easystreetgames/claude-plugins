# Journal Plugin

A conversational journaling assistant for Claude Code. It interviews you about your day with genuine curiosity, then writes a polished Markdown journal entry to `documents/journal/YYYY-MM-DD.md`.

## Install

Copy [command.md](command.md) to `.claude/commands/journal.md` in any project (or `~/.claude/commands/journal.md` for global access).

## Usage

```
/journal
```

Claude will ask you a series of open-ended questions about your day, follow up on interesting threads, and then write the entry when the conversation feels complete.

## Output

Entries are saved to:

```
documents/journal/YYYY-MM-DD.md
```

Each entry includes a title, date, narrative prose, and a brief reflection section.
