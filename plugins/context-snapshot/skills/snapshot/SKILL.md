---
name: snapshot
description: Rebuild current_context.md from all available artifacts (journal entries, Linear board, usability reports, research notes). Use when context has drifted, after a break, or to sync the context file with recent work. Triggers on "context snapshot", "refresh context", "rebuild context", "update current_context", "my context is stale", or any request to regenerate the standing context file.
---

# Context Snapshot

You are rebuilding `current_context.md` — the standing context file that `prioritize:now` and the journal skill read at the start of every session. Your job is to synthesize all available artifacts into a fresh, accurate snapshot of the user's current projects, status, and people.

## Journal Path (silent — do not narrate this step)

Before any file operations, determine the journal folder:
1. Read `.claude/journal-path.txt` in the current working directory.
2. If found, use that path as `JOURNAL_PATH`.
3. If not found, use `./journal` as `JOURNAL_PATH`.
4. Create `JOURNAL_PATH` if it does not exist.

## Step 1 — Load artifacts (silent — do not narrate this to the user)

Gather everything before speaking:

1. **Recent journals**: Read the last 14 days of `{JOURNAL_PATH}/YYYY-MM-DD.md` files (skip missing dates). For each, extract: projects mentioned, people, unchecked todos (`- [ ]`), checked todos (`- [x]`), blockers, and any status updates.

2. **Existing context file**: Read `{JOURNAL_PATH}/current_context.md`. Treat this as a starting point — it may be stale but it's the best baseline you have.

3. **Linear board**: Linear is the only task source; do not read `task:*` or `project:*` entities from the MCP memory graph. Read-only: this skill never writes to Linear.

   | Workspace | MCP server | Scope |
   |-----------|-----------|-------|
   | EasyStreetGames (EAS) | `linear-esg` | team **EasyStreetGames** |
   | Intusic (INT) | `linear` | issues assigned to `me` |

   Call `list_issues` on each source with `fields: ["title", "status", "statusType", "project", "updatedAt", "completedAt", "url"]`, once for In Progress (`state: "started"`), once for Ready (`state: "unstarted"`), and once for recently done (`completedAt: "-P14D"`). Group cards by their Linear `project`. Skip a source silently if its server isn't connected.

4. **Log file**: Read `{JOURNAL_PATH}/log.md` if present for recent blockers or mid-session notes.

5. **Usability reports**: Look for files matching `{JOURNAL_PATH}/usability-report-*.md` and `{JOURNAL_PATH}/usability-*.md`. For any found, note the subject, date, and any severity-3 or severity-4 findings that haven't been resolved.

6. **Research notes**: Look for files in `{JOURNAL_PATH}/research/` and matching `{JOURNAL_PATH}/research-*.md`. For any found within the last 30 days, note the topic and whether it produced a concrete recommendation.

7. **Insights** (optional): Skim `{JOURNAL_PATH}/insights/*.md` from the last 30 days for patterns that should appear under open threads.

If no artifacts at all exist, tell the user and stop — there is nothing to synthesize.

## Step 2 — Synthesize

From the loaded data, reconstruct the user's current picture:

- **Active work**: What projects are actively in progress? What shipped? What's next?
- **Stalled or blocked work**: Work that is open in journals or Linear (In Progress or Ready) but hasn't been touched recently
- **People**: Who is actively involved in each project right now? Omit people not mentioned in recent journals
- **Open UX or research threads**: Active usability reports or research topics that are influencing current work
- **Technical**: The tools and stacks currently in use across projects

**Conflict resolution**: If Linear and the journals disagree about a card's status, report Linear's status and note the journal's claim in one line; Linear is the task source.

**Recency filter**: Remove details from the existing context file that haven't appeared in the last 14 days of journals and have no open Linear cards — they're likely stale.

## Step 3 — Draft

Write a new `current_context.md` using this exact format:

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

Overwrite `{JOURNAL_PATH}/current_context.md` with the approved content. Confirm success in one sentence.
