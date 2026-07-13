# introspection

A reflective conversation partner that loads your journal history and priority graph to surface work patterns, stalled tasks, and recurring themes — then talks through them with you one at a time.

## Install

```text
/plugin install introspection@esgames-plugins
```

## Usage

| Command | When to use |
| ------- | ----------- |
| `/introspection:introspect` | Zoom out on your work — weekly reflection, feeling stuck, or noticing something is off |

## What it does

On invoke, it silently reads the last 14 days of journal entries and your MCP memory graph, then opens with the most interesting pattern it finds — a stalled task, a priority mismatch, a recurring blocker, or a dropped thread.

It asks one question at a time and follows the conversation wherever you take it. When you end the session, it writes any confirmed insights and task updates back to the memory graph.

## Dependencies

- **Journal files**: `~/Downloads/Journal/YYYY-MM-DD.txt` — reads the last 14 days
- **MCP memory graph**: reads tasks, projects, and prior insights; writes updates as the conversation unfolds. Falls back to journal-only if the graph is unavailable.

## Works well with

```text
/journal:journal         ← capture today before reflecting on the week
/prioritize:now          ← act on what the reflection surfaces
```
