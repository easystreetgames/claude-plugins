---
description: Pick what to work on next by energy, from your Linear board and the repo's work queue. Shows the queue's next step plus In Progress and Ready cards (unranked), asks how much energy you have, and suggests cards that match. To refill the queue, use prioritize:queue. Use when the user says things like "what should I work on", "help me plan my day", "I need to prioritize", "what can I pull", or "triage my tasks".
---

# Pull by Energy

Work is pulled kanban-style by current energy, not ranked. The board is never ordered by priority and the skill never outputs a "Top 3". It shows what is in flight and what is ready, asks how much energy the user has right now, and suggests cards that fit.

The one ordered thing is the repo's work queue (`documents/WORK-QUEUE.md`), which the user owns. This skill shows it as it stands and never edits it; `prioritize:queue` refills it.

Linear and the work queue are the only task sources. Do not read or write `task:*` entities in the MCP memory graph, and do not read journal todo lists as tasks.

## Sources

| Workspace | MCP server | Scope | Access |
|-----------|-----------|-------|--------|
| EasyStreetGames (EAS) | `linear-esg` | team **EasyStreetGames** | read; write only with consent (see step 5) |
| Intusic (INT) | `linear` | issues assigned to `me` | **read-only**, never write |

If one server is not connected, say so in one line and continue with the other. If neither is, say that the skill needs a Linear MCP server and stop.

**Work queue.** If the current repo has `documents/WORK-QUEUE.md`, read it. It lists EAS issues only. Its own "Instructions for Claude" section is the authority on how rows, statuses and Linear order work. This skill only reads it.

## Steps

### 1. Load the board

Call `list_issues` on each source, once per state type, asking only for the fields you need (`title`, `status`, `statusType`, `estimate`, `labels`, `project`, `url`):

- **In Progress**: `state: "started"`
- **Ready**: `state: "unstarted"` (EAS calls it Todo)

Use state *types*, not names, so both workspaces work. Leave out Backlog, Done, Canceled and Duplicate, except for issues the work queue names (below).

For each Ready card, check whether it is blocked: call `get_issue` with `includeRelations: true` and look at `blockedBy`. A card whose blocker is not Done or Canceled moves to **Waiting On** with the blocker named. Skip this check for In Progress cards.

If there is a work queue, take from it:

- the `▶` step (the first Queue row not `done`), with its type and status, or "queue empty" if there is none;
- the other Queue rows that are not `done` or `skipped`, in file order;
- the **Not yet ordered** issues.

Fetch any of these that the Linear calls above didn't return (usually Backlog issues) with `get_issue`, for their title, status and estimate. Each queue issue appears once on the board, in the Queue column, not again under In Progress or Ready.

### 2. Show the board

Present the board grouped by column. The Queue column keeps the file's order; every other column is unranked. Within a column, group by workspace (EAS, then INT). Do not number cards or sort them by priority.

```
## Board

### Queue
- ▶ EAS-212 fix: Script ratings 521–661 (in progress)
- EAS-213 build: Town Day text pass
- Not yet ordered: EAS-165, EAS-115, EAS-179 (Backlog)

### In Progress
- EAS-16 Rethink prioritize so it reads Linear
- INT-42 Instrument representation

### Ready
- EAS-201 Town Day: canvas text at least 12px

### Waiting On
- EAS-210 Deploy step — waiting on EAS-209
```

If the `▶` row is `awaiting approval`, say so on its line: the next move is the user's review, not new work. If the queue is empty, say so in one line. Omit an empty column. If In Progress has more than three cards, say so in one line: finishing something may beat pulling something new.

### 3. Ask about energy

Ask one question and wait for the answer:

*"How much energy do you have right now: low, medium, or high? (Or tell me how long you've got.)"*

### 4. Suggest matching cards

Size each card from what Linear says about it: its `estimate` if set, otherwise its labels, title and (if needed) its description. Then match:

- **Low**: small, mechanical or well-specified work: a fix with a known cause, docs, a close-out, a review, finishing a nearly done In Progress card.
- **Medium**: a normal build step with a clear spec.
- **High**: design work, open-ended problems, anything new or hard to start.

Suggest one to three cards that fit. Order of preference when more than one fits: the `▶` queue step (say "next" works it), then In Progress cards (finishing beats starting), then the rest. Later Queue rows and Not yet ordered issues are not workable through "next" until they reach `▶` or are moved into the queue, so say so if you suggest one. For each, give one sentence on why it matches the stated energy. If nothing fits, say so and name the closest card.

```
## Fits low energy

- **EAS-201 Town Day: canvas text at least 12px** — a known-cause layout fix with a clear check at the end.
- **INT-42 Instrument representation** — already in progress; the next step is small.
```

Keep the reasons specific to the card. No generic productivity advice.

### 5. Pull (optional)

Ask: *"Want to pull one?"* If the user picks a **Ready EAS card**, offer to set it to In Progress with `save_issue` (`state: "In Progress"`) on `linear-esg`, and do it only on a yes. Never change priorities, estimates, relations, or anything on INT.

If the user reports something mid-day (finished a card, hit a blocker, new work came in), tell them what to change in Linear in one line each. For EAS cards, offer to make the change; for INT, the user makes it.

### 6. Refill the queue

If the queue is empty or nearly empty (one row left that isn't `done`), suggest `/prioritize:queue` in one line. Don't refill it here.
