# context-snapshot

Rebuilds `current_context.md` — the standing context file read by `prioritize:now` and the journal skill — from all available artifacts.

## When to use

- After a long break when `current_context.md` has gone stale
- When you've been journaling regularly but context has drifted from reality
- After completing a major project phase (shipped, stalled, pivoted)
- Any time you want to force a clean sync of all plugin artifacts

## What it reads

| Artifact | Source |
|----------|--------|
| Recent journal entries | `{JOURNAL_PATH}/YYYY-MM-DD.md` (last 14 days) |
| Linear board | `linear-esg` (EAS) and `linear` (INT, assigned to you): In Progress, Ready, and done in the last 14 days. Read-only |
| Log file | `{JOURNAL_PATH}/log.md` |
| Insights | `{JOURNAL_PATH}/insights/*.md` |
| Usability reports | `{JOURNAL_PATH}/usability-report-*.md` |
| Research notes | `{JOURNAL_PATH}/research/` and `research-*.md` |

Missing artifacts are skipped gracefully — the skill works with whatever is available.

## What it produces

An updated `{JOURNAL_PATH}/current_context.md` with:
- Active project status (what shipped, what's in progress, what's next)
- Current people and their roles
- Active technical stacks
- Any open UX or research threads shaping current work

Stale entries — things not mentioned in the last 14 days and not open in Linear — are pruned automatically.

## Usage

```
/context-snapshot:snapshot
```

The skill shows a draft and a change summary before writing. You approve before anything is saved.

## Setup

No additional setup required. Optionally pairs with:
- `prioritize` — reads the same Linear board
- `journal` — both read and write `current_context.md`
- `introspection` — reads the same journal entries
