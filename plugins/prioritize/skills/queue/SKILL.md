---
description: Propose a prioritized refill or reorder of the repo's work queue (documents/WORK-QUEUE.md) from Linear, and write it only after the user approves. Use when the user says things like "refill the queue", "plan the queue", "what should go in the queue next", "reorder the queue", or when the queue is empty.
---

# Refill the Work Queue

The work queue (`documents/WORK-QUEUE.md`) is the one ordered list of work, and the user owns it. This skill proposes the next batch of steps and their order, and changes nothing until the user approves.

The file's own "Instructions for Claude" section is the authority on rows, step numbers, statuses and Linear order. Follow it whenever this skill edits the file. If the repo has no `documents/WORK-QUEUE.md`, say so and stop.

Use the `linear-esg` MCP only (team **EasyStreetGames**). The queue lists EAS issues only; never read or write another workspace here.

## Steps

### 1. Read the queue

Read `WORK-QUEUE.md`: the Queue rows that are not `done` or `skipped`, the `▶` step, the Order paragraph, the Not yet ordered table and its notes, and Handoff.

### 2. Gather candidates

Candidates are the Not yet ordered issues, plus Todo and In Progress EAS cards that aren't in the queue (`list_issues` with `state: "unstarted"` and `state: "started"`). For each one, call `get_issue` with `includeRelations: true` and read its description, status, estimate, `blockedBy` and `blocks`.

Notes in the queue file outrank Linear text. A note like "Rob plays levels 3–5 first" or "Eric's to fix" means the issue isn't ready to queue.

### 3. Propose an order

Propose five to eight steps at most, as a table in chat:

| # | Issue | Type | Step | Why here |
|---|---|---|---|---|

- Blockers come first, then unblocked work that unblocks others, then work that continues something just finished.
- An issue may need more than one step (design, then build). Keep a design step and its build step apart, since design stops at `awaiting approval`.
- Types are the file's: `lesson`, `design`, `build`, `fix`, `close`, `other`.
- Give each row a one-sentence reason for its place.

Below the table, list what you left out and why (waiting on someone, too vague to start, superseded). If the queue still has open rows, say whether the new steps go after them or whether you suggest a reorder, and why.

### 4. Wait for approval

Ask the user to approve the proposal or change it. Write nothing before that. Revise and re-propose as often as they ask.

### 5. Write it

On approval:

1. Add the rows to the Queue table. Step numbers continue the file's count, status is `todo`, and `▶` goes on the first row that isn't `done`.
2. Remove the queued issues from Not yet ordered.
3. Update **Current step** and the **Order** paragraph (what this batch is and today's date).
4. Run the file's **Linear order** sync on `linear-esg` (priorities and blocks). Approving the refill counts as the user asking for it.
5. Commit the file only if the user asks.

End with one line: how many steps were queued and what `▶` is now.
