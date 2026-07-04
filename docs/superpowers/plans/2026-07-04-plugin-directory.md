# Plugin Directory Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a standalone `esg-plugin-directory` repo with a `/directory:refresh` skill that fetches the community plugin catalog, categorizes plugins with AI, captures interest ratings per category, and writes a curated `DIRECTORY.md`.

**Architecture:** A Claude Code plugin repo with a single skill (`refresh`) that acts as a conversational workflow — reads a remote GitHub catalog, reads local `interests.json` for saved preferences, interactively asks about new or all categories, then writes updated `interests.json` and a curated `DIRECTORY.md`. All file I/O uses absolute paths so the skill works regardless of which project the user invokes it from.

**Tech Stack:** Claude Code plugin manifest (JSON), SKILL.md prompt, Markdown, JSON. Skill uses WebFetch (catalog retrieval), Read, and Write tools at runtime.

## Global Constraints

- All paths in SKILL.md must be absolute — `~/Documents/GitHub/esg-plugin-directory/` — not relative
- Plugin name: `directory` (skill invoked as `/directory:refresh`)
- Plugin manifest `$schema`: `https://www.schemastore.org/claude-code-plugin-manifest.json`
- No automated commits — user reviews and commits manually
- Catalog source: `jeremylongshore/claude-code-plugins-plus-skills`
- Repo location: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\`

---

## File Map

| File | Responsibility |
|------|---------------|
| `.claude-plugin/plugin.json` | Plugin manifest — name, version, skills path |
| `skills/refresh/SKILL.md` | The refresh skill prompt — full conversational workflow |
| `interests.json` | Persisted category preferences — read and written by the skill |
| `DIRECTORY.md` | Curated output — written by the skill, committed by the user |
| `README.md` | Usage docs — how to install, invoke, configure |
| `.gitignore` | Standard ignores |

---

### Task 1: Initialize the repo

**Files:**
- Create: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\.gitignore`

**Interfaces:**
- Produces: a git repo at `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\`

- [ ] **Step 1: Create directory and initialize git**

```powershell
New-Item -ItemType Directory -Path "C:\Users\Rob\Documents\GitHub\esg-plugin-directory"
Set-Location "C:\Users\Rob\Documents\GitHub\esg-plugin-directory"
git init
```

Expected: `Initialized empty Git repository in C:/Users/Rob/Documents/GitHub/esg-plugin-directory/.git/`

- [ ] **Step 2: Write .gitignore**

Write to `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\.gitignore`:

```
.DS_Store
Thumbs.db
```

- [ ] **Step 3: Initial commit**

```powershell
git add .gitignore
git commit -m "chore: initialize repo"
```

---

### Task 2: Plugin manifest and stub files

**Files:**
- Create: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\.claude-plugin\plugin.json`
- Create: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\interests.json`
- Create: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\DIRECTORY.md`

**Interfaces:**
- Produces: a valid Claude Code plugin manifest, making the repo installable with `/plugin install`

- [ ] **Step 1: Create manifest directory**

```powershell
New-Item -ItemType Directory -Path "C:\Users\Rob\Documents\GitHub\esg-plugin-directory\.claude-plugin"
```

- [ ] **Step 2: Write plugin.json**

Write to `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\.claude-plugin\plugin.json`:

```json
{
  "$schema": "https://www.schemastore.org/claude-code-plugin-manifest.json",
  "name": "directory",
  "description": "Curated directory of Claude Code plugins. Fetches the community catalog, categorizes plugins with AI, and generates a DIRECTORY.md based on your interests.",
  "version": "1.0.0",
  "skills": "./skills"
}
```

- [ ] **Step 3: Write stub interests.json**

Write to `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\interests.json`:

```json
{}
```

An empty object signals first-run state. The skill writes the full structure after the first interactive session.

- [ ] **Step 4: Write stub DIRECTORY.md**

Write to `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\DIRECTORY.md`:

```markdown
# Claude Plugin Directory

_Run `/directory:refresh` to generate this file._
```

- [ ] **Step 5: Commit**

```powershell
git add .claude-plugin\plugin.json interests.json DIRECTORY.md
git commit -m "feat: add plugin manifest and stub files"
```

---

### Task 3: Write the refresh skill

**Files:**
- Create: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\skills\refresh\SKILL.md`

**Interfaces:**
- Consumes: `~/Documents/GitHub/esg-plugin-directory/interests.json` (empty `{}` on first run)
- Produces: updated `~/Documents/GitHub/esg-plugin-directory/interests.json` and `~/Documents/GitHub/esg-plugin-directory/DIRECTORY.md`

- [ ] **Step 1: Create skills directory**

```powershell
New-Item -ItemType Directory -Path "C:\Users\Rob\Documents\GitHub\esg-plugin-directory\skills\refresh" -Force
```

- [ ] **Step 2: Write SKILL.md**

Write to `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\skills\refresh\SKILL.md`:

````markdown
---
description: Fetch the community plugin catalog, categorize plugins with AI, capture your interests, and write a curated DIRECTORY.md.
---

# Plugin Directory Refresh

This skill fetches all plugins from the community Claude Code catalog, organizes them into categories, asks you to rate your interest in each, and writes a curated `DIRECTORY.md` to the directory repo.

## Paths

All files live at `~/Documents/GitHub/esg-plugin-directory/`. If you cloned the repo elsewhere, update these two paths before using the skill.

- Preferences: `~/Documents/GitHub/esg-plugin-directory/interests.json`
- Output: `~/Documents/GitHub/esg-plugin-directory/DIRECTORY.md`

---

## Step 1 — Fetch the community catalog

Use WebFetch to retrieve the catalog. Try these URLs in order until one returns usable content:

1. `https://raw.githubusercontent.com/jeremylongshore/claude-code-plugins-plus-skills/main/README.md`
2. `https://github.com/jeremylongshore/claude-code-plugins-plus-skills`

Parse all plugin entries from the response. For each plugin, extract:
- Name
- Marketplace / source identifier
- Description (infer from surrounding context if not explicit)
- Source URL (link to the plugin or its source repo)

Note the total plugin count.

If neither URL returns usable content, stop and tell the user: "Could not fetch the community catalog. Check your internet connection or verify the source URL."

---

## Step 2 — Read interests.json

Read `~/Documents/GitHub/esg-plugin-directory/interests.json`.

- File is empty (`{}`) or missing → **first run**, no saved preferences
- File contains a `categories` object → **subsequent run**, load those preferences

---

## Step 3 — Categorize all plugins

Analyze all plugin names and descriptions. Assign each plugin to exactly one category. Derive category names from the content — do not use a prescribed list.

Guidelines:
- Aim for 8–15 distinct categories
- Name categories specifically enough to be useful ("Game Dev Tools", "Writing Assistants") but broad enough that each has at least 3 entries
- Derive the JSON key as the kebab-case version of the name ("Game Dev Tools" → `game-dev-tools`)

---

## Step 4 — Capture interest ratings

### First run

Present all proposed categories as a numbered list with plugin counts. Ask the user to rate each one: **interested**, **not interested**, or **maybe**.

```
I've organized [N] plugins into [M] categories. Please rate your interest in each
(interested / not interested / maybe):

1. Dev Tools (42 plugins)
2. Writing Assistants (38 plugins)
3. Game Dev Tools (12 plugins)
...
```

Wait for the user's full response before proceeding.

### Subsequent run

Compare current proposed categories against the keys in `interests.json`.

- Already rated → use saved rating, do not re-ask
- New categories (not in interests.json) → present only these for rating using the same numbered list format

If there are no new categories: "No new categories since last refresh. Using saved preferences." Then skip to Step 5.

---

## Step 5 — Write interests.json

Merge new ratings with any existing ones. Write to `~/Documents/GitHub/esg-plugin-directory/interests.json`:

```json
{
  "last_updated": "YYYY-MM-DD",
  "catalog_source": "jeremylongshore/claude-code-plugins-plus-skills",
  "categories": {
    "dev-tools": "interested",
    "writing-assistants": "not interested",
    "game-dev-tools": "maybe"
  }
}
```

Use today's date for `last_updated`. Category keys are kebab-case.

---

## Step 6 — Generate DIRECTORY.md

Write to `~/Documents/GitHub/esg-plugin-directory/DIRECTORY.md`.

Include only `interested` and `maybe` categories. Omit `not interested` entirely.

**Order:**
1. `interested` categories — alphabetical, each as a `##` section
2. `maybe` categories — all under a single `## Maybe Interesting` section, each as a `###` sub-header

**Format:**

```markdown
# Claude Plugin Directory

_Last updated: YYYY-MM-DD · N plugins across community marketplaces_

## [Category Name]

| Plugin | Marketplace | Description |
|--------|-------------|-------------|
| [plugin-name](source-url) | marketplace-name | One-line description |

## Maybe Interesting

### [Category Name]

| Plugin | Marketplace | Description |
|--------|-------------|-------------|
| [plugin-name](source-url) | marketplace-name | One-line description |
```

- Plugin name is a markdown link to the plugin's source URL
- Descriptions trimmed to one line
- Marketplace is the identifier from the community catalog

---

## Done

Tell the user:
- How many plugins are included (interested + maybe categories combined)
- How many were omitted (not interested categories)
- That `interests.json` and `DIRECTORY.md` have been written and are ready to review and commit
````

- [ ] **Step 3: Verify frontmatter**

Open `skills\refresh\SKILL.md` and confirm the first three lines are:
```
---
description: Fetch the community plugin catalog, categorize plugins with AI, capture your interests, and write a curated DIRECTORY.md.
---
```
The `---` delimiters must be on their own lines with no leading whitespace.

- [ ] **Step 4: Commit**

```powershell
git add skills\refresh\SKILL.md
git commit -m "feat: add directory refresh skill"
```

- [ ] **Step 5: Install and verify the plugin loads**

From any Claude Code project, run:
```
/plugin install directory --local C:\Users\Rob\Documents\GitHub\esg-plugin-directory
```

Then invoke:
```
/directory:refresh
```

Expected: skill loads, fetches the catalog, presents plugin categories for rating. If the skill fails to load, confirm `plugin.json`'s `"skills": "./skills"` path resolves correctly relative to the manifest.

---

### Task 4: Write README.md

**Files:**
- Create: `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\README.md`

**Interfaces:**
- Produces: user-facing installation and usage documentation

- [ ] **Step 1: Write README.md**

Write to `C:\Users\Rob\Documents\GitHub\esg-plugin-directory\README.md`:

```markdown
# esg-plugin-directory

A curated directory of Claude Code plugins from across community marketplaces. Fetches 425+ plugins, organizes them by category using AI, and generates a `DIRECTORY.md` based on your interests.

## Installing

From a local clone:
```
/plugin install directory --local /path/to/esg-plugin-directory
```

Once published to GitHub:
```
/plugin marketplace add https://github.com/easystreetgames/esg-plugin-directory
/plugin install directory@esg-plugin-directory
```

## Usage

Invoke the refresh skill whenever you want to update the directory:

```
/directory:refresh
```

**First run:** the skill asks you to rate interest in each plugin category (interested / not interested / maybe).

**Subsequent runs:** only asks about categories that are new since your last refresh.

Results are saved to:

- `interests.json` — your category preferences (hand-editable anytime)
- `DIRECTORY.md` — the curated plugin list

Review both files and commit when satisfied.

## Data source

Community catalog: [jeremylongshore/claude-code-plugins-plus-skills](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)

## Path configuration

The skill writes to `~/Documents/GitHub/esg-plugin-directory/` by default. If you cloned this repo to a different location, update the paths in `skills/refresh/SKILL.md`.
```

- [ ] **Step 2: Commit**

```powershell
git add README.md
git commit -m "docs: add README"
```
