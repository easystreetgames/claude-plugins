# prioritize

Pull-by-energy work picker for Claude Code. Reads your Linear board, shows In Progress and Ready cards unranked, asks how much energy you have, and suggests cards that match.

## Setup

Connect the Linear MCP servers the skill reads:

- `linear-esg`: the EasyStreetGames workspace (team **EasyStreetGames**). Read, and written only when you say yes.
- `linear`: the Intusic workspace. Read-only; only issues assigned to you.

Either one alone works; the skill says which is missing. The MCP memory graph is no longer used.

## Usage

```
/prioritize:now
```

1. Loads In Progress (`started`) and Ready (`unstarted`, EAS's Todo) cards. Ready cards with an open blocker move to **Waiting On**.
2. Shows the board grouped by column, never numbered or sorted by priority.
3. Asks: low, medium or high energy (or how long you've got).
4. Suggests one to three cards that fit, finishing In Progress work first, with a one-line reason each.
5. Offers to move a pulled EAS card to In Progress. INT is never written.

## Changes in 2.0.0

- No ranking, no morning kickoff, no "Today's Top 3". Work is pulled by energy.
- Tasks come from Linear, not `task:*` entities in the memory graph or journal todos.
