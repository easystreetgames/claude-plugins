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

On invoke, it silently reads the last 14 days of journal entries, `log.md`, prior files under `insights/`, and your lean MCP memory graph, then opens with the most interesting pattern it finds — a stalled task, a priority mismatch, a recurring blocker, or a dropped thread.

It asks one question at a time and follows the conversation wherever you take it. When you end the session, it writes confirmed insights to `{journal-path}/insights/YYYY-MM-DD-{slug}.md` and applies lean task updates to the graph (status/priority only; narrative goes to `log.md`).

## Dependencies

- **Journal files**: `YYYY-MM-DD.md` — reads the last 14 days
- **`log.md`**: mid-session notes and blocker detail
- **`insights/`**: prior reflection notes (file-based, not stored as heavy graph entities)
- **MCP memory graph**: lean index of open tasks and projects. Falls back to journal-only if the graph is unavailable.

## Works well with

```text
/journal:journal         ← capture today before reflecting on the week
/prioritize:now          ← act on what the reflection surfaces
```
