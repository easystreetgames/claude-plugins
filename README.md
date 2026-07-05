# Claude Plugin Marketplace

A collection of Claude Code plugins — custom slash commands you can drop into any project.

## Installing a Plugin

```text
/plugin marketplace add https://github.com/easystreetgames/claude-plugins
/plugin install journal@esgames-plugins
```

Then invoke the skill:

```text
/journal:journal
```

## Plugins

| Plugin | Skill | Description |
| ------ | ----- | ----------- |
| [journal](plugins/journal/) | `/journal:journal` | Conversational journaling assistant — interviews you about your day and saves a Markdown entry |
| [prioritize](plugins/prioritize/) | `/prioritize:now` | Morning kickoff or mid-day triage — reads your journal todos and updates a priority graph |
| [introspection](plugins/introspection/) | `/introspection:introspect` | Reflective conversation partner — surfaces work patterns and stalled themes from your journal history and priority graph |
| [research](plugins/research/) | `/research:research` | Iterative research — broad sweep then user-directed deep dives, each in a clean subagent |
| [usability-report](plugins/usability-report/) | `/usability-report:nielsen-review` | Heuristic evaluation against Nielsen's 10 principles — outputs a severity-rated Markdown report |
| [context-snapshot](plugins/context-snapshot/) | `/context-snapshot:snapshot` | Rebuilds the standing context file from all artifacts — journal entries, priority graph, usability reports, and research notes |

## Workflows

The plugins are designed to compose. Here are the natural combinations.

### Daily work rhythm

```text
/prioritize:now          ← morning: reads yesterday's journal todos, sets top 3 priorities
  ... do work ...
/prioritize:now          ← mid-day: triage what changed, refocus
/journal:journal         ← end of day: capture what happened + tomorrow's todos
```

The journal's todo list becomes the next morning's kickoff input automatically.

### Weekly reflection

```text
/introspection:introspect
```

Loads your full journal history and priority graph and surfaces recurring themes, stalled work, and patterns you might not notice day-to-day. Run it Friday afternoon or Monday morning when you want to zoom out.

### Researching before you build

```text
/research:research       ← broad sweep on the topic, then deep dives on what matters
/journal:journal         ← capture what you learned and any todos it generated
/prioritize:now          ← triage the new work against existing priorities
```

Use this when exploring an unfamiliar library, architectural approach, or domain before committing to an implementation path.

### UX review → prioritized fixes

```text
/usability-report:nielsen-review <url-or-path>   ← generates severity-rated findings
/prioritize:now                                   ← triage severity 3–4 issues into your day
/journal:journal                                  ← record what you fixed and what's still open
```

The usability report gives you concrete severity scores; prioritize helps you decide which fires to fight today.

### Recovering from context drift

Run this after a break, after switching machines, or when `prioritize:now` or `journal:journal` seem to be working from stale information:

```text
/context-snapshot:snapshot   ← reads last 14 days of journals, priority graph,
                                usability reports, and research notes — synthesizes
                                a fresh current_context.txt for approval before writing
```

All other skills read `current_context.txt` at startup. Running this brings them back in sync.

### Full product loop

```text
/research:research           ← explore the problem space
/usability-report:...        ← evaluate the current state of the UI
/journal:journal             ← capture findings and next steps
/prioritize:now              ← set priorities from those todos
/introspection:introspect    ← zoom out after a few weeks to check direction
```

## Contributing

Each plugin lives in `plugins/<name>/` with:

- `README.md` — what it does and how to use it
- `.claude-plugin/plugin.json` — plugin manifest (name, version, skills paths)
- `skills/<skill-name>/SKILL.md` — the skill prompt with frontmatter description
