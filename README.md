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
| [prioritize](plugins/prioritize/) | `/prioritize:now` | Pull work by energy — shows your Linear In Progress and Ready cards, unranked, and suggests ones that fit |
| [introspection](plugins/introspection/) | `/introspection:introspect` | Reflective conversation partner — surfaces work patterns and stalled themes from your journal history and priority graph |
| [research](plugins/research/) | `/research:research` | Iterative research — broad sweep then user-directed deep dives, each in a clean subagent |
| [usability-report](plugins/usability-report/) | `/usability-report:nielsen-review` | Heuristic evaluation against Nielsen's 10 principles — outputs a severity-rated Markdown report |
| [context-snapshot](plugins/context-snapshot/) | `/context-snapshot:snapshot` | Rebuilds the standing context file from all artifacts — journal entries, priority graph, usability reports, and research notes |
| [one-on-one](plugins/one-on-one/) | `/one-on-one:add` `/one-on-one:review` `/one-on-one:debrief` | Running topic lists for 1:1 meetings — capture topics as they come up, pull an agenda before the meeting, mark items discussed after |

## Workflows

The plugins are designed to compose. Here are the natural combinations.

### Daily work rhythm

```text
/prioritize:now          ← any time: see your Linear board, pull a card that fits your energy
  ... do work ...
/prioritize:now          ← when a card finishes or blocks: pull the next one
/journal:journal         ← end of day: capture what happened + tomorrow's todos
```

Tasks live in Linear; the journal records the day around them.

### Weekly reflection

```text
/introspection:introspect
```

Loads your full journal history and priority graph and surfaces recurring themes, stalled work, and patterns you might not notice day-to-day. Run it Friday afternoon or Monday morning when you want to zoom out.

### Researching before you build

```text
/research:research       ← broad sweep on the topic, then deep dives on what matters
/journal:journal         ← capture what you learned and any todos it generated
/prioritize:now          ← pull the new work when your energy fits it
```

Use this when exploring an unfamiliar library, architectural approach, or domain before committing to an implementation path.

### UX review → prioritized fixes

```text
/usability-report:nielsen-review <url-or-path>   ← generates severity-rated findings
/prioritize:now                                   ← pull a severity 3–4 fix once it's a Linear card
/journal:journal                                  ← record what you fixed and what's still open
```

The usability report gives you concrete severity scores; prioritize helps you pick which fix fits your energy.

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
/prioritize:now              ← pull the next card from Linear
/introspection:introspect    ← zoom out after a few weeks to check direction
```

## Contributing

Each plugin lives in `plugins/<name>/` with:

- `README.md` — what it does and how to use it
- `.claude-plugin/plugin.json` — plugin manifest (name, version, skills paths)
- `skills/<skill-name>/SKILL.md` — the skill prompt with frontmatter description

### Dev setup

After cloning, activate the pre-commit hook that enforces version bumps:

```bash
git config core.hooksPath .githooks
```

The hook blocks commits that modify plugin files without bumping the version in `plugin.json`.
