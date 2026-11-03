# introspection

A reflective conversation partner that loads your journal history and Linear board to surface work patterns, stalled tasks, and recurring themes — then talks through them with you one at a time.

## Install

```text
/plugin install introspection@esgames-plugins
```

## Usage

| Command | When to use |
| ------- | ----------- |
| `/introspection:introspect` | Zoom out on your work — weekly reflection, feeling stuck, or noticing something is off |

## What it does

On invoke, it silently reads the last 14 days of journal entries, `log.md`, prior files under `insights/`, and your Linear cards (In Progress, Ready, and done in the last 14 days), then opens with the most interesting pattern it finds — a stalled task, a priority mismatch, a recurring blocker, or a dropped thread.

It asks one question at a time and follows the conversation wherever you take it. When you end the session, it writes confirmed insights to `{journal-path}/insights/YYYY-MM-DD-{slug}.md` and logs confirmed stalls and blockers to `log.md`. If a pattern implies a Linear change, it lists it; it makes EAS changes only on a yes and never writes to INT.

## Dependencies

- **Journal files**: `YYYY-MM-DD.md` — reads the last 14 days
- **`log.md`**: mid-session notes and blocker detail
- **`insights/`**: prior reflection notes (file-based)
- **Linear MCP servers**: `linear-esg` (EAS team) and `linear` (INT, issues assigned to you, read-only). Falls back to journal-only if neither is connected.

## Works well with

```text
/journal:journal         ← capture today before reflecting on the week
/prioritize:now          ← act on what the reflection surfaces
```
