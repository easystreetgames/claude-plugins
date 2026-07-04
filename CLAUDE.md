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

1. Create `plugins/<name>/` with the structure above.
2. `plugin.json` minimum fields: `name`, `description`, `version`, `skills` (path to skills dir).
3. `SKILL.md` requires YAML frontmatter with a `description` field; the body is the prompt:

   ```yaml
   ---
   description: One-line description shown in skill picker
   ---
   ```

4. Register the plugin in `.claude-plugin/marketplace.json` under `plugins[]` with a `name` and `"source": "./plugins/<name>"`.
5. Bump `version` in `plugin.json` on every release — Claude Code uses this to detect updates.

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
