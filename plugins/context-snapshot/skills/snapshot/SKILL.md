---
name: snapshot
description: Rebuild current_context.txt from all available artifacts (journal entries, priority graph, usability reports, research notes). Use when context has drifted, after a break, or to sync the context file with recent work. Triggers on "context snapshot", "refresh context", "rebuild context", "update current_context", "my context is stale", or any request to regenerate the standing context file.
---

# Context Snapshot

You are rebuilding `current_context.txt` — the standing context file that `prioritize:now` and the journal skill read at the start of every session. Your job is to synthesize all available artifacts into a fresh, accurate snapshot of the user's current projects, status, and people.

## Journal Path (silent — do not narrate this step)

Before any file operations, determine the journal folder:
1. Read `.claude/journal-path.txt` in the current working directory.
2. If found, use that path as `JOURNAL_PATH`.
3. If not found, use `./journal` as `JOURNAL_PATH`.
4. Create `JOURNAL_PATH` if it does not exist.

## Step 1 — Load artifacts (silent — do not narrate this to the user)

Gather everything before speaking:

1. **Recent journals**: Read the last 14 days of `{JOURNAL_PATH}/YYYY-MM-DD.txt` files (skip missing dates). For each, extract: projects mentioned, people, unchecked todos (`- [ ]`), checked todos (`- [x]`), blockers, and any status updates.

2. **Existing context file**: Read `{JOURNAL_PATH}/current_context.txt`. Treat this as a starting point — it may be stale but it's the best baseline you have.

3. **Priority graph**: Call `read_graph`. Collect all `task:*` and `project:*` entities with their `status`, `priority`, and `note` observations. Note which tasks are done vs. still open. Skip silently if unavailable.

4. **Usability reports**: Look for files matching `{JOURNAL_PATH}/usability-report-*.md` and `{JOURNAL_PATH}/usability-*.md`. For any found, note the subject, date, and any severity-3 or severity-4 findings that haven't been resolved.

5. **Research notes**: Look for files in `{JOURNAL_PATH}/research/` and matching `{JOURNAL_PATH}/research-*.md`. For any found within the last 30 days, note the topic and whether it produced a concrete recommendation.

If no artifacts at all exist, tell the user and stop — there is nothing to synthesize.

## Step 2 — Synthesize

From the loaded data, reconstruct the user's current picture:

- **Active work**: What projects are actively in progress? What shipped? What's next?
- **Stalled or blocked work**: Tasks that appear open in journals or the graph but haven't been touched recently
- **People**: Who is actively involved in each project right now? Omit people not mentioned in recent journals
- **Open UX or research threads**: Active usability reports or research topics that are influencing current work
- **Technical**: The tools and stacks currently in use across projects

**Conflict resolution**: If the graph and journals disagree about a task's status, trust the most recent journal entry over the graph.

**Recency filter**: Remove details from the existing context file that haven't appeared in the last 14 days of journals and have no open graph entries — they're likely stale.

## Step 3 — Draft

Write a new `current_context.txt` using this exact format:

```
# Current Context
Last Updated: {YYYY-MM-DD}

## Work: {Primary Work Context} ({Role or Capacity})

- **{Area or subproject}**: {Current status — 1–3 sentences. What shipped, what's active, what's coming next.}
[... one bullet per active area ...]

## Side Project: {Name}

- {Current status — active participants, what's in progress or stalled, what's next.}
[... one bullet per significant dimension of this project ...]

[... additional ## Side Project sections as needed ...]

## Key People

- **{Name}** ({role/org}): {Current engagement — one sentence.}
[... one line per person actively mentioned in recent journals ...]

## Technical

- {Primary work stack and environment}
- {Side project stacks in active use}
[... one bullet per distinct technical area — omit inactive stacks ...]
```

Rules:
- Only include what you can derive from the loaded artifacts — don't invent status
- Keep each bullet current and specific, not a history lesson
- If usability reports or research notes are actively shaping current work, add a brief `## Open Threads` section with one line per active thread
- Omit entire sections if you have no recent data for them

## Step 4 — Present for approval

Show the full draft. Below it, give a brief change summary:

- **Added**: new sections or entries not in the old file
- **Updated**: entries that changed materially
- **Removed**: stale entries that no longer appear in recent data

Then ask: *"Does this look right, or anything to adjust before I write it?"*

Apply any requested changes and re-confirm. Do not write the file until the user approves.

## Step 5 — Write

Overwrite `{JOURNAL_PATH}/current_context.txt` with the approved content. Confirm success in one sentence.
