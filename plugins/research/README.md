# research

Iterative research using subagents. Start with a broad overview, then steer Claude into deep dives on the aspects that matter to you.

## Install

```text
/plugin install research@esgames-plugins
```

## Usage

```text
/research:research
```

Then just tell Claude what topic to research. It will:

1. Run a broad sweep via subagent and return 5–8 numbered aspects
2. Ask which aspect(s) you want to explore deeper
3. Dispatch a focused subagent for each deep dive
4. Repeat until you say "done", then offer a short synthesis

## Why subagents?

Each research phase runs in a dedicated subagent, so the main context stays clean no matter how many deep dives you do. You can explore 10 aspects without context bloat.
