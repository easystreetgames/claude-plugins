# one-on-one

Manage running topic lists for your one-on-one meetings. Capture things to discuss with specific people as they come up, then pull up a clean agenda right before the meeting.

## Install

```text
/plugin install one-on-one@esgames-plugins
```

## Usage

| Command               | When to use                              |
| --------------------- | ---------------------------------------- |
| `/one-on-one:add`     | Capture a topic to discuss with someone  |
| `/one-on-one:review`  | Pull up your agenda before a meeting     |
| `/one-on-one:debrief` | Mark topics as discussed after a meeting |

## Storage

Topics are persisted in the MCP memory graph (same graph used by `journal` and `prioritize`). If MCP is unavailable, topics are shown in-session only.

## Entity schema

| Entity | Pattern                    |
| ------ | -------------------------- |
| Person | `person:{name}`            |
| Topic  | `1on1-topic:{name}:{slug}` |
