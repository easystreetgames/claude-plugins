# Plugin Directory — Design Spec — 2026-07-04

## Problem

The Claude Code plugin ecosystem has grown to 425 plugins and 2,810 skills across community marketplaces. There is no quality signal or discovery layer — just a flat list. Users have no way to find what is worth using.

## Solution

A standalone plugin repo (`esg-plugin-directory`) that ships a `/directory:refresh` skill. When invoked, the skill fetches the community catalog, uses AI to categorize all plugins, captures the user's interest in each category, and writes a curated `DIRECTORY.md`. Category preferences persist in `interests.json` so subsequent refreshes are fast — only new categories require input.

## Repo Structure

```text
esg-plugin-directory/
  .claude-plugin/plugin.json     ← plugin manifest
  skills/refresh/SKILL.md        ← /directory:refresh skill
  interests.json                 ← persisted category preferences (committed)
  DIRECTORY.md                   ← curated output (committed)
  README.md
```

The repo is itself a Claude plugin — anyone can install it and get the skill.

## Refresh Flow

When `/directory:refresh` is invoked:

1. Fetch the community catalog from `jeremylongshore/claude-code-plugins-plus-skills`
2. Read `interests.json` (absent on first run)
3. Use AI to propose category names and assign all plugins to them (names are not prescribed — the AI derives them from the catalog content)
4. **First run:** present all categories and ask the user to rate each one (`interested` / `not interested` / `maybe`)
5. **Subsequent runs:** load saved preferences; only ask about categories new since last refresh
6. Write updated `interests.json`
7. Generate `DIRECTORY.md` from high-interest and maybe categories

The skill runs entirely in-conversation. No commits are made automatically — the user reviews and commits when satisfied.

## Data Formats

### interests.json

```json
{
  "last_updated": "2026-07-04",
  "catalog_source": "jeremylongshore/claude-code-plugins-plus-skills",
  "categories": {
    "productivity": "interested",
    "dev-tools": "interested",
    "writing": "not interested",
    "data-analysis": "maybe"
  }
}
```

Hand-editable at any time. The skill reads this before running and writes to it after capturing new ratings.

### DIRECTORY.md

```markdown
# Claude Plugin Directory
_Last updated: 2026-07-04 · 425 plugins across community marketplaces_

## Dev Tools
| Plugin | Marketplace | Description |
|--------|-------------|-------------|
| plugin-name | marketplace-name | One-line description |

## Productivity
...

## Maybe Interesting
...
```

- Sections ordered: `interested` categories first, `maybe` at the bottom
- `not interested` categories are omitted entirely
- Each row: plugin name linked to source, marketplace name, one-line description
- No scoring or star ratings — curation by category is the signal

## Out of Scope

- Automated/scheduled updates (user invokes manually)
- Quality scoring within categories
- A web-based UI
- Modifying or extending the source community catalog
