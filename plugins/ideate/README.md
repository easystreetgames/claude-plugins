# ideate

Brainstorm a topic with a panel of subagents. One generates ideas, three topic-specific personas score them independently, and you get the list ranked by normalized score.

## Install

```text
/plugin install ideate@esgames-plugins
```

## Usage

```text
/ideate:ideate <topic>
```

For example: `/ideate:ideate retention mechanics for a daily solitaire game`. Leave the topic off and Claude will ask for one.

Claude will:

1. Generate ~12 ideas and design 3 diverse personas for the topic (parallel subagents)
2. Have each persona score every idea 1–10 with a one-line reason (parallel subagents, blind to each other)
3. Normalize each persona's scores to 0–10 so harsh and generous graders count equally, then average them
4. Present the ideas in score order with per-persona columns, notes on the top 3, and the most divisive idea

## Why normalize?

One persona might score everything 6–8 while another uses 2–9. Min-max scaling each persona's scores before averaging keeps any single reviewer's grading habits from dominating the ranking.
