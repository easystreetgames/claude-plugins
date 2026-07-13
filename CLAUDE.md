# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A Claude Code plugin marketplace. The marketplace is registered as `esgames-plugins`. Users add it with:

```text
/plugin marketplace add <repo-url>
/plugin install <plugin-name>@esgames-plugins
```

## Repository structure

```text
.claude-plugin/marketplace.json        ← marketplace catalog (registry of all plugins)
plugins/<name>/
  .claude-plugin/plugin.json           ← plugin manifest (name, version, skills path)
  skills/<skill-name>/SKILL.md         ← skill prompt with YAML frontmatter
  README.md                            ← user-facing docs for this plugin
README.md                              ← marketplace overview
```

## Adding a new plugin

### 1. Create the plugin directory

```text
plugins/<name>/
  .claude-plugin/plugin.json
  skills/<skill-name>/SKILL.md   ← one directory per skill/slash command
  README.md
```

### 2. Write `plugin.json`

Minimum required fields:

```json
{
  "$schema": "https://www.schemastore.org/claude-code-plugin-manifest.json",
  "name": "my-plugin",
  "description": "One-line description of what this plugin does.",
  "version": "1.0.0",
  "skills": "./skills"
}
```

Bump `version` on every release — Claude Code uses this to detect updates.

### 3. Write each `SKILL.md`

Each subdirectory under `skills/` becomes one slash command: `/<plugin-name>:<skill-dir-name>`.

`SKILL.md` requires a YAML frontmatter `description`; the rest is the prompt body:

```yaml
---
description: One-line description shown in skill picker
---

# Skill title

Your full system prompt goes here.
```

A plugin with multiple skills gets multiple directories — one per command:

```text
skills/
  add/SKILL.md      → /my-plugin:add
  review/SKILL.md   → /my-plugin:review
  debrief/SKILL.md  → /my-plugin:debrief
```

To remove a skill, delete its directory.

### 4. Write `README.md`

User-facing docs for the plugin. Cover install command, each slash command with a one-line description, and any storage or dependencies (e.g. MCP memory graph).

### 5. Register in the marketplace

Add an entry to `.claude-plugin/marketplace.json` under `plugins[]`:

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin",
  "description": "Same one-liner as plugin.json."
}
```

## Testing locally

Install a plugin from a local path and invoke its skill to verify:

```text
/plugin install <name> --local ./plugins/<name>
/<plugin-name>:<skill-name>
```

## Naming rules

- Marketplace `name` (in `marketplace.json`) must not contain "claude", "anthropic", or any string that could be mistaken for an official Anthropic marketplace. Current name: `esgames-plugins`.
- Plugin and skill names use kebab-case, no spaces.
- Skills are invoked as `/<plugin-name>:<skill-name>`.

## Schema references

- Marketplace: `https://www.schemastore.org/claude-code-marketplace.json`
- Plugin manifest: `https://www.schemastore.org/claude-code-plugin-manifest.json`
- Full docs: `https://code.claude.com/docs/en/plugin-marketplaces`
