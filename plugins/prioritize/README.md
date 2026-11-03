# prioritize

Pull-by-energy work picker for Claude Code. Reads your Linear board and the repo's `documents/WORK-QUEUE.md` (if any), shows the queue's next step plus In Progress and Ready cards unranked, asks how much energy you have, and suggests cards that match.

## Setup

Connect the Linear MCP servers the skill reads:

- `linear-esg`: the EasyStreetGames workspace (team **EasyStreetGames**). Read, and written only when you say yes.
- `linear`: the Intusic workspace. Read-only; only issues assigned to you.

Either one alone works; the skill says which is missing. The MCP memory graph is no longer used.

## Usage

```
/prioritize:now
```

1. Loads In Progress (`started`) and Ready (`unstarted`, EAS's Todo) cards. Ready cards with an open blocker move to **Waiting On**. If the repo has `documents/WORK-QUEUE.md`, also loads its `▶` step, open Queue rows and Not yet ordered issues (Backlog included).
2. Shows the board grouped by column. The Queue column keeps the file's order; the rest is never numbered or sorted by priority.
3. Asks: low, medium or high energy (or how long you've got).
4. Suggests one to three cards that fit, finishing In Progress work first, with a one-line reason each.
5. Offers to move a pulled EAS card to In Progress. INT is never written.
6. Points you to `/prioritize:queue` when the queue is empty or nearly so.

```
/prioritize:queue
```

Proposes a prioritized refill (or reorder) of `documents/WORK-QUEUE.md` from the Not yet ordered issues and open EAS cards, with a reason for each place. On your approval it writes the rows and syncs Linear priorities and blocks by the file's own rules.

## Changes in 2.1.0

- Reads `documents/WORK-QUEUE.md`: a Queue column, the `▶` step suggested first when it fits, and Backlog issues the queue names.
- New `/prioritize:queue`: propose a prioritized queue refill, written only on approval.

## Changes in 2.0.0

- No ranking, no morning kickoff, no "Today's Top 3". Work is pulled by energy.
- Tasks come from Linear, not `task:*` entities in the memory graph or journal todos.
