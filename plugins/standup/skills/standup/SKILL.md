---
description: Generate a concise morning standup report — Yesterday / Today / Blockers — pulled from your journal and priority graph. Team-facing, short, no fluff.
---

# Standup Report

You are generating a morning standup report. Output must be short, concrete, and team-facing — only things teammates care about. No personal reflections, no context-setting, no filler.

## Step 1 — Load artifacts (silent — do not narrate)

1. **Yesterday's journal**: Compute yesterday's date (today minus 1 weekday — if today is Monday, use Friday). Read `~/Downloads/Journal/{yesterday}.txt`. Extract:
   - Completed work: checked todos (`- [x]`), and things described as done, shipped, finished, discussed, reviewed, prepared, or tested
   - Skip purely personal or administrative entries (lunch, breaks, filing) unless directly team-relevant

2. **Today's plan**: Read `~/Downloads/Journal/{today}.txt` if it exists — extract unchecked todos (`- [ ]`). If no today journal exists, read `~/Downloads/Journal/current_context.txt` and infer today's intended work from active project status and open tasks.

3. **Priority graph**: Call `read_graph`. Collect:
   - Tasks with `status: blocked` or a `note:` observation that describes an external dependency or blocker
   - Tasks with `priority: 1` and `status: pending` (supplement Today if not already captured from the journal)
   - Skip silently if unavailable

If no artifacts exist at all, tell the user and stop.

## Step 2 — Synthesize

**Yesterday** — 2–5 bullets of what was actually done. Use action verbs: prepared, shipped, reviewed, met with, fixed, discussed, tested. Name the project or person when it adds clarity. Drop anything vague ("worked on X") — be specific about the outcome.

**Today** — 2–4 bullets of what is planned. Draw from today's unchecked journal todos and priority-1 graph tasks. Focus on outcomes and deliverables, not busywork.

**Blockers** — 0–3 bullets of things stopped or waiting on something outside your control. If nothing is blocked, omit this section entirely — do not write "None".

Rules:
- Max 5 bullets per section
- Each bullet is one line; no sub-bullets
- No preamble or trailing commentary in the report itself
- Strip bullets that are purely internal or not meaningful to a team

## Step 3 — Output

Print the report in this exact format:

```
**Yesterday**
- [bullet]
- [bullet]

**Today**
- [bullet]
- [bullet]

**Blockers**
- [bullet]
```

Omit the Blockers section entirely if there are none.

After the report, add one blank line, then ask: *"Anything to add or adjust?"* Apply any changes and reprint the updated report.
