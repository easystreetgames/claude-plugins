---
description: Pick what to work on next by energy, from your Linear board. Shows In Progress and Ready cards (unranked), asks how much energy you have, and suggests cards that match. Use when the user says things like "what should I work on", "help me plan my day", "I need to prioritize", "what can I pull", or "triage my tasks".
---

# Pull by Energy

Work is pulled kanban-style by current energy, not ranked. This skill never orders cards by priority and never outputs a "Top 3". It shows what is in flight and what is ready, asks how much energy the user has right now, and suggests cards that fit.

Linear is the only task source. Do not read or write `task:*` entities in the MCP memory graph, and do not read journal todo lists as tasks.

## Sources

| Workspace | MCP server | Scope | Access |
|-----------|-----------|-------|--------|
| EasyStreetGames (EAS) | `linear-esg` | team **EasyStreetGames** | read; write only with consent (see step 5) |
| Intusic (INT) | `linear` | issues assigned to `me` | **read-only**, never write |

If one server is not connected, say so in one line and continue with the other. If neither is, say that the skill needs a Linear MCP server and stop.

## Steps

### 1. Load the board

Call `list_issues` on each source, once per state type, asking only for the fields you need (`title`, `status`, `statusType`, `estimate`, `labels`, `project`, `url`):

- **In Progress**: `state: "started"`
- **Ready**: `state: "unstarted"` (EAS calls it Todo)

Use state *types*, not names, so both workspaces work. Leave out Backlog, Done, Canceled and Duplicate.

For each Ready card, check whether it is blocked: call `get_issue` with `includeRelations: true` and look at `blockedBy`. A card whose blocker is not Done or Canceled moves to **Waiting On** with the blocker named. Skip this check for In Progress cards.

### 2. Show the board

Present the board grouped by column, unranked. Within a column, group by workspace (EAS, then INT). Do not number cards or sort them by priority.

```
## Board

### In Progress
- EAS-16 Rethink prioritize so it reads Linear
- INT-42 Instrument representation

### Ready
- EAS-201 Town Day: canvas text at least 12px

### Waiting On
- EAS-210 Deploy step — waiting on EAS-209
```

Omit an empty column. If In Progress has more than three cards, say so in one line: finishing something may beat pulling something new.

### 3. Ask about energy

Ask one question and wait for the answer:

*"How much energy do you have right now: low, medium, or high? (Or tell me how long you've got.)"*

### 4. Suggest matching cards

Size each card from what Linear says about it: its `estimate` if set, otherwise its labels, title and (if needed) its description. Then match:

- **Low**: small, mechanical or well-specified work: a fix with a known cause, docs, a close-out, a review, finishing a nearly done In Progress card.
- **Medium**: a normal build step with a clear spec.
- **High**: design work, open-ended problems, anything new or hard to start.

Suggest one to three cards that fit, In Progress cards first when one fits (finishing beats starting). For each, give one sentence on why it matches the stated energy. If nothing fits, say so and name the closest card.

```
## Fits low energy

- **EAS-201 Town Day: canvas text at least 12px** — a known-cause layout fix with a clear check at the end.
- **INT-42 Instrument representation** — already in progress; the next step is small.
```

Keep the reasons specific to the card. No generic productivity advice.

### 5. Pull (optional)

Ask: *"Want to pull one?"* If the user picks a **Ready EAS card**, offer to set it to In Progress with `save_issue` (`state: "In Progress"`) on `linear-esg`, and do it only on a yes. Never change priorities, estimates, relations, or anything on INT.

If the user reports something mid-day (finished a card, hit a blocker, new work came in), tell them what to change in Linear in one line each. For EAS cards, offer to make the change; for INT, the user makes it.

---

## Appendix: Memory Graph Hygiene (canonical)

This skill no longer uses the graph. The rules stay here because `journal`, `introspection` and `one-on-one` follow them by name. The graph is a **lean index** — identity, latest status/priority, relations, optional file refs. Narrative lives in journal files (`YYYY-MM-DD.md`, `log.md`, `current_context.md`, `insights/`). All graph-writing skills follow these rules:

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
