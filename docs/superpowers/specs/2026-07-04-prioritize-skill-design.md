# Prioritize Skill — Design Spec
**Date:** 2026-07-04  
**Status:** Approved

---

## Overview

A new `prioritize` plugin with a single skill `/prioritize:now` that helps the user decide what to work on at any point in the day. It uses `@modelcontextprotocol/server-memory` as a local knowledge graph to maintain context across invocations, and reads the journal skill's output as its starting data source.

---

## Plugin Structure

```
plugins/prioritize/
  .claude-plugin/plugin.json
  skills/now/SKILL.md
  README.md
```

Registered in `.claude-plugin/marketplace.json` under `plugins[]` as `prioritize`.

---

## MCP Dependency

The skill requires `@modelcontextprotocol/server-memory`. The SKILL.md opens with a **Setup** block containing the exact config snippet to add to `~/.claude/settings.json`:

```json
"mcpServers": {
  "memory": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-memory"],
    "env": {
      "MEMORY_FILE_PATH": "C:/Users/Rob/Downloads/Journal/memory/priority_graph.json"
    }
  }
}
```

The graph file is co-located with the journal data at `~/Downloads/Journal/memory/priority_graph.json`. On invoke, the skill verifies MCP tools are available (by attempting `read_graph`) and halts with setup instructions if they are not.

---

## Graph Schema

### Entities

| Type | Name pattern | Purpose |
|------|-------------|---------|
| `session` | `session:YYYY-MM-DD` | Marks that morning kickoff occurred today |
| `task` | `task:{kebab-slug}` | Individual work items |
| `project` | `project:{kebab-slug}` | Ongoing projects (solitaire, intusic, esg, etc.) |

### Observations (on task entities)

| Key | Values |
|-----|--------|
| `priority` | `1`, `2`, `3`, ... |
| `status` | `pending`, `in_progress`, `done`, `blocked` |
| `note` | Free-form context string (optional) |

### Relations

| From | Relation | To |
|------|----------|----|
| `task` | `belongs_to` | `project` |
| `task` | `blocks` | `task` |

---

## Mode Detection

On every invocation, the skill calls `search_nodes` for `session:{today's date}`.

- **Not found** → Morning kickoff mode
- **Found** → Triage mode

No prompt to the user. Detection is silent and automatic.

---

## Morning Kickoff Flow

1. Read `~/Downloads/Journal/YYYY-MM-DD.txt` (today's date) if it exists
2. Read `~/Downloads/Journal/current_context.txt` for standing project context
3. Extract unchecked items from the journal's `## Todo` section; create `task` entities for each, linked to their project where identifiable
4. Create `session:{today}` entity to record kickoff
5. Ask conversationally:
   - *"What's the most important thing you need to finish today?"*
   - *"Anything new, blocking, or competing for your attention?"*
6. Upsert tasks with `priority` and `status` observations; add relations
7. Output: ranked top-3 tasks with a one-sentence rationale per item

---

## Triage Flow

1. Call `read_graph` / `search_nodes` to load today's tasks, statuses, and blockers
2. Ask conversationally:
   - *"What just happened — did you finish something, get interrupted, or hit a blocker?"*
   - *"Anything new that came in?"*
3. Update graph: mark tasks done, add new task entities, record blockers as `note` observations or `blocks` relations
4. Output: one clear next-step recommendation with brief reasoning — *"Based on what's changed, here's what I'd focus on next and why."*

---

## Journal Integration

- **Reads** today's journal file (`YYYY-MM-DD.txt`) and `current_context.txt` during morning kickoff
- **Does not write** to journal files — that remains the journal skill's responsibility
- If today's journal doesn't exist yet, the skill falls back to `current_context.txt` alone and relies on the opening conversation to surface today's tasks

---

## Error Handling

| Condition | Behavior |
|-----------|----------|
| MCP tools not available | Halt; print setup instructions from the Setup block |
| No journal file for today | Continue with `current_context.txt` only |
| No `current_context.txt` | Continue; morning kickoff relies entirely on conversation |
| Graph already has today's session but is empty | Treat as triage, ask what the user is working on |

---

## Out of Scope (v1)

- Writing back to journal files
- Richer graph schema (typed relations beyond `belongs_to` / `blocks`)
- Multi-day trend analysis or reporting
- Web search or external context (that belongs to the journal skill)
