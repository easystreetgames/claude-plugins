# context-snapshot

Rebuilds `current_context.txt` — the standing context file read by `prioritize:now` and the journal skill — from all available artifacts.

## When to use

- After a long break when `current_context.txt` has gone stale
- When you've been journaling regularly but context has drifted from reality
- After completing a major project phase (shipped, stalled, pivoted)
- Any time you want to force a clean sync of all plugin artifacts

## What it reads

| Artifact | Source |
|----------|--------|
| Recent journal entries | `~/Downloads/Journal/YYYY-MM-DD.txt` (last 14 days) |
| Priority graph | MCP memory server (`read_graph`) |
| Usability reports | `~/Downloads/Journal/usability-report-*.md` |
| Research notes | `~/Downloads/Journal/research/` and `research-*.md` |

Missing artifacts are skipped gracefully — the skill works with whatever is available.

## What it produces

An updated `~/Downloads/Journal/current_context.txt` with:
- Active project status (what shipped, what's in progress, what's next)
- Current people and their roles
- Active technical stacks
- Any open UX or research threads shaping current work

Stale entries — things not mentioned in the last 14 days and not open in the graph — are pruned automatically.

## Usage

```
/context-snapshot:snapshot
```

The skill shows a draft and a change summary before writing. You approve before anything is saved.

## Setup

No additional setup required. Optionally pairs with:
- `prioritize` — shares the same MCP memory graph
- `journal` — both read and write `current_context.txt`
- `introspection` — reads the same journal entries
