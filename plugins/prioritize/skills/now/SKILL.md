---
description: Prioritize your work at any point in the day. Runs morning kickoff or mid-day triage automatically based on today's memory graph state. Use when the user says things like "what should I work on", "help me plan my day", "I need to prioritize", "morning kickoff", or "triage my tasks".
---

# Work Prioritizer

## Setup (required before first use)

This skill requires the `@modelcontextprotocol/server-memory` MCP server. Create or edit `~/.claude/.mcp.json` (your global MCP config) and add the `memory` entry, then restart Claude Code:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "env": {
        "MEMORY_FILE_PATH": "~/Downloads/Journal/memory/priority_graph.json"
      }
    }
  }
}
```

> **Note:** `~` expands to your home directory. The `memory/` subdirectory will be created automatically on first use. You only need to do this setup once.

---

## On Every Invocation

**Step 1 — Verify MCP availability**

Call `read_graph`. If the call fails or the tool is not available, display the full Setup section above verbatim, then offer to continue in **no-graph mode**: run the same Morning Kickoff or Triage conversation without persisting anything to the graph. Skip all `create_entities`, `create_relations`, and `add_observations` calls in no-graph mode, and tell the user their priorities won't be remembered across sessions until setup is complete.

**Step 2 — Get today's date**

Determine today's date in `YYYY-MM-DD` format.

**Step 3 — Detect mode**

Call `search_nodes` with the query `session:{today}` (e.g. `session:2026-07-04`).

- No matching entity found → run **Morning Kickoff** below
- Matching entity found → run **Triage** below

No need to tell the user which mode was detected. Just run it.

---

## Morning Kickoff

You are helping the user start their workday with clarity and a concrete priority order.

### 1. Load context

- Attempt to read `~/Downloads/Journal/{today}.txt`. If it exists, extract every unchecked item (`- [ ]`) from the Todo section — match any heading level and capitalization (`## Todo`, `### Todo`, `## TODO`, etc.).
- Attempt to read `~/Downloads/Journal/current_context.txt` for standing project and people context.
- If neither file exists, proceed with an empty task list and rely entirely on conversation.

### 2. Initialize the session

Call `create_entities` with:
```json
[{ "name": "session:{today}", "entityType": "session", "observations": ["started: {ISO timestamp}"] }]
```

### 3. Seed the graph from journal todos

For each unchecked todo item extracted in step 1:

1. Generate a kebab-slug from the task text. Keep it short and meaningful. Examples:
   - "Track outcome of Andrew/Kat Solitaire review" → `track-kat-solitaire-review`
   - "Continue Intusic prototype: instrument representation" → `intusic-instrument-representation`

2. Call `create_entities`:
   ```json
   [{ "name": "task:{slug}", "entityType": "task", "observations": ["status: pending"] }]
   ```

3. If you can identify the relevant project from `current_context.txt` (e.g. Solitaire → `project:solitaire`, Intusic → `project:intusic`, ESG → `project:esg`), call `create_entities` for that project entity if it doesn't already exist, then call `create_relations`:
   ```json
   [{ "from": "task:{slug}", "to": "project:{project-slug}", "relationType": "belongs_to" }]
   ```

### 4. Converse

Ask one question at a time. Wait for the full response before asking the next.

1. *"What's the most important thing you need to finish today?"*
2. *"Anything new, blocking, or competing for your attention?"*

Based on what the user shares, create or update task entities. Assign `priority: 1`, `priority: 2`, `priority: 3` observations to the tasks that matter most. Add `note: {context}` observations for blockers or dependencies the user mentions. Add `blocks` relations where one task explicitly holds up another:
```json
[{ "from": "task:{blocker-slug}", "to": "task:{blocked-slug}", "relationType": "blocks" }]
```

### 5. Output: Today's Top 3

Present this block at the end of the conversation:

```
## Today's Top 3

1. **[Task name]** — [One sentence: why this is #1 given what the user just said]
2. **[Task name]** — [One sentence: why this is #2]
3. **[Task name]** — [One sentence: why this is #3]
```

Keep rationale specific and grounded in what the user told you. No generic productivity advice.

---

## Triage

You are helping the user re-orient after something changed mid-day.

### 1. Load current state

Call `read_graph`. Identify all task entities. Note which have `status: done`, `status: in_progress`, `status: blocked`, or `status: pending`. Note any `blocks` relations between tasks.

### 2. Converse

Ask one question at a time. Wait for the full response before asking the next.

1. *"What just happened — did you finish something, get interrupted, or hit a blocker?"*
2. *"Anything new that came in?"*

### 3. Update the graph

Based on what the user tells you:

- **Task finished:** call `add_observations`:
  ```json
  [{ "entityName": "task:{slug}", "contents": ["status: done"] }]
  ```
- **New task:** call `create_entities` with `status: pending` observation, then `create_relations` for project link (same pattern as Morning Kickoff step 3)
- **Blocker:** call `add_observations` with `note: {blocker description}` in `contents`, and `create_relations` with `relationType: blocks` if one task is holding up another:
  ```json
  [{ "entityName": "task:{slug}", "contents": ["note: {blocker description}"] }]
  ```
- **Reprioritize:** call `add_observations` to update `priority` values on affected tasks:
  ```json
  [{ "entityName": "task:{slug}", "contents": ["priority: 1"] }]
  ```

### 4. Output: What to focus on next

Pick the highest-priority non-done, non-blocked task given what just changed. Present:

```
## What to focus on next

**[Task name]**

[2–3 sentences: why this is the right next thing given what the user just told you. Be specific about what changed. No generic advice.]
```

---

## Graph Schema Reference

| Entity | Name pattern | Observations |
|--------|-------------|--------------|
| Session | `session:YYYY-MM-DD` | `started: {timestamp}` |
| Task | `task:{kebab-slug}` | `priority: 1\|2\|3...`, `status: pending\|in_progress\|done\|blocked`, `note: {text}` |
| Project | `project:{kebab-slug}` | — |

| Relation | Meaning |
|----------|---------|
| `task → belongs_to → project` | Task belongs to a project |
| `task → blocks → task` | One task is blocking another |
