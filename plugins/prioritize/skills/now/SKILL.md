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
        "MEMORY_FILE_PATH": "{your-journal-path}/memory/priority_graph.json"
      }
    }
  }
}
```

> **Note:** Replace `{your-journal-path}` with the path from `/journal:path` (e.g. `C:\Users\Rob\Documents\GitHub\my-project\Journal`). The `memory/` subdirectory will be created automatically on first use. You only need to do this setup once.

---

## Memory Graph Hygiene (canonical)

The graph is a **lean index** — identity, latest status/priority, relations, optional file refs. Narrative lives in journal files (`YYYY-MM-DD.md`, `log.md`, `current_context.md`, `insights/`). All graph-writing skills follow these rules:

1. **Replace, don't append** — Before writing a new `status:` or `priority:`, call `delete_observations` for any existing observations on that entity that start with the same key (`status:`, `priority:`). Then `add_observations` with the new value. Never leave multiple statuses/priorities on one entity.

2. **No free-form notes in the graph** — Do not write `note: {paragraph}`. Instead:
   - Mid-session / triage detail → append a line to `{JOURNAL_PATH}/log.md` (format: `[YYYY-MM-DD HH:MM] {text}`).
   - Standing context → `current_context.md` or the daily journal, not the graph.
   - If a pointer is needed: one observation `ref: YYYY-MM-DD.md` or `ref: log.md` (relative to `JOURNAL_PATH`).

3. **Prune closed entities** — Use `delete_entities` (relations go away with the entity):
   - **Tasks** whose latest `status: done` is not from today's work → delete on morning kickoff (and journal MCP sync).
   - **Sessions** `session:*` where date &lt; today → delete on every invoke.
   - **1:1 topics** marked discussed → delete immediately in debrief (do not keep `status: discussed`).
   - **Insights** → files under `{JOURNAL_PATH}/insights/`, not heavy graph observations.

4. **Open set only** — After hygiene, the graph should mostly hold open work: pending/in_progress/blocked tasks, active projects, pending 1:1 topics, today's session.

**Helper — set status or priority (use every time):**

```
1. From read_graph / known state, collect existing observations on the entity that start with "status:" or "priority:" (whichever key you are updating).
2. If any exist, call delete_observations with those exact observation strings.
3. Call add_observations with the new value only.
```

**Helper — log narrative (use instead of note:):**

Append to `{JOURNAL_PATH}/log.md`:
```
[YYYY-MM-DD HH:MM] {concise detail}
```

---

## On Every Invocation

**Step 1 — Verify MCP availability**

Call `read_graph`. If the call fails or the tool is not available, display the full Setup section above verbatim, then offer to continue in **no-graph mode**: run the same Morning Kickoff or Triage conversation without persisting anything to the graph. Skip all graph write/prune calls in no-graph mode, and tell the user their priorities won't be remembered across sessions until setup is complete.

**Step 2 — Get today's date and journal path**

Determine today's date in `YYYY-MM-DD` format.

**Journal Path**: Read `.claude/journal-path.txt` in the current working directory. If found, use that path as `JOURNAL_PATH`; otherwise use `./journal`.

**Step 3 — Prune (skip in no-graph mode)**

From the graph just read:

1. Call `delete_entities` for every `session:*` entity whose date is before today.
2. Call `delete_entities` for every `task:*` whose **latest** `status:` observation is `done` (stale closed work — do not keep done tasks across days).

Do not narrate prune steps to the user.

**Step 4 — Detect mode**

Call `search_nodes` with the query `session:{today}` (e.g. `session:2026-07-04`).

- No matching entity found → run **Morning Kickoff** below
- Matching entity found → run **Triage** below

No need to tell the user which mode was detected. Just run it.

---

## Morning Kickoff

You are helping the user start their workday with clarity and a concrete priority order.

### 1. Load context

- Attempt to read `{JOURNAL_PATH}/{today}.md`. If it exists, extract every unchecked item (`- [ ]`) from the Todo section — match any heading level and capitalization (`## Todo`, `### Todo`, `## TODO`, etc.).
- Attempt to read `{JOURNAL_PATH}/current_context.md` for standing project and people context.
- Attempt to read `{JOURNAL_PATH}/log.md` for mid-day notes not yet journaled.
- If neither journal nor context exists, proceed with an empty task list and rely entirely on conversation.

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

3. If you can identify the relevant project from `current_context.md` (e.g. Solitaire → `project:solitaire`, Intusic → `project:intusic`, ESG → `project:esg`), call `create_entities` for that project entity if it doesn't already exist, then call `create_relations`:
   ```json
   [{ "from": "task:{slug}", "to": "project:{project-slug}", "relationType": "belongs_to" }]
   ```

### 4. Converse

Ask one question at a time. Wait for the full response before asking the next.

1. *"What's the most important thing you need to finish today?"*
2. *"Anything new, blocking, or competing for your attention?"*

Based on what the user shares:

- Create or update task entities (lean observations only).
- Set `priority: 1`, `priority: 2`, `priority: 3` using the **replace, don't append** helper.
- For blockers or dependencies: append detail to `{JOURNAL_PATH}/log.md`; set `status: blocked` via the replace helper when a task is blocked; add `blocks` relations where one task explicitly holds up another:
```json
[{ "from": "task:{blocker-slug}", "to": "task:{blocked-slug}", "relationType": "blocks" }]
```
- Optionally set `ref: log.md` on a blocked task if a pointer helps; never write free-form `note:` observations.

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

Call `read_graph` (after prune). Identify open task entities. Note which have `status: in_progress`, `status: blocked`, or `status: pending`. Note any `blocks` relations. Read `{JOURNAL_PATH}/log.md` if present for recent narrative.

### 2. Converse

Ask one question at a time. Wait for the full response before asking the next.

1. *"What just happened — did you finish something, get interrupted, or hit a blocker?"*
2. *"Anything new that came in?"*

### 3. Update the graph

Based on what the user tells you:

- **Task finished:** Prefer `delete_entities` for `task:{slug}` (remove finished work from the graph). If you need to keep it visible only for the rest of today, set `status: done` via the replace helper — morning prune will remove it tomorrow.
- **New task:** call `create_entities` with `status: pending` observation, then `create_relations` for project link (same pattern as Morning Kickoff step 3).
- **Blocker:** append `[YYYY-MM-DD HH:MM] {blocker description}` to `{JOURNAL_PATH}/log.md`. Set `status: blocked` via the replace helper. Add `blocks` relations if one task holds up another. Optionally `ref: log.md`. Never write `note:`.
- **Reprioritize:** set `priority:` via the replace helper:
  ```json
  [{ "entityName": "task:{slug}", "contents": ["priority: 1"] }]
  ```
  (after deleting any prior `priority:*` observations)

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
| Task | `task:{kebab-slug}` | **one** `priority: 1\|2\|3...`, **one** `status: pending\|in_progress\|done\|blocked`, optional `ref: {path}` |
| Project | `project:{kebab-slug}` | — |

| Relation | Meaning |
|----------|---------|
| `task → belongs_to → project` | Task belongs to a project |
| `task → blocks → task` | One task is blocking another |

Do **not** store free-form `note:` observations. Detail goes in `{JOURNAL_PATH}/log.md` or journal files.
