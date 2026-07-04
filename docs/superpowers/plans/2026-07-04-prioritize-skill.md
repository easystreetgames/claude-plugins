# Prioritize Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a `/prioritize:now` plugin skill that uses `@modelcontextprotocol/server-memory` to run morning kickoff or mid-day triage based on today's graph state, reading journal output as its seed data.

**Architecture:** Single plugin (`prioritize`) with one skill (`now`). The SKILL.md prompt drives all behavior — mode detection, graph reads/writes, and conversational flows. No code beyond the prompt file and plugin manifest.

**Tech Stack:** Claude Code plugin system, `@modelcontextprotocol/server-memory` (npx-runnable), journal skill output files (`~/Downloads/Journal/`)

## Global Constraints

- Plugin name must be kebab-case, no spaces, not "claude" or "anthropic"
- SKILL.md requires YAML frontmatter with `description` field
- plugin.json must include `name`, `description`, `version`, `skills` fields
- Skill invocation: `/prioritize:now`
- Graph file path: `C:/Users/Rob/Downloads/Journal/memory/priority_graph.json`
- Journal files: `~/Downloads/Journal/YYYY-MM-DD.txt` and `~/Downloads/Journal/current_context.txt`
- The skill must NOT write to journal files
- MCP check must be the first action on every invocation; halt with setup instructions if unavailable

---

### Task 1: Plugin scaffold and marketplace registration

**Files:**
- Create: `plugins/prioritize/.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

**Interfaces:**
- Produces: a valid plugin manifest that Claude Code can load; a marketplace entry pointing to `./plugins/prioritize`

- [ ] **Step 1: Create plugin.json**

Create `plugins/prioritize/.claude-plugin/plugin.json` with this exact content:

```json
{
  "$schema": "https://www.schemastore.org/claude-code-plugin-manifest.json",
  "name": "prioritize",
  "description": "Work prioritizer using a local MCP memory graph. Runs morning kickoff or mid-day triage based on today's graph state.",
  "version": "1.0.0",
  "skills": "./skills"
}
```

- [ ] **Step 2: Register in marketplace.json**

Open `.claude-plugin/marketplace.json`. Add this object to the end of the `plugins` array:

```json
{
  "name": "prioritize",
  "source": "./plugins/prioritize",
  "description": "Work prioritizer using a local MCP memory graph. Runs morning kickoff or mid-day triage based on today's graph state."
}
```

The final `plugins` array should have three entries: `journal`, `usability-report`, and `prioritize`.

- [ ] **Step 3: Verify JSON validity**

Run:
```powershell
python3 -c "import json; json.load(open('.claude-plugin/marketplace.json')); print('OK')"
python3 -c "import json; json.load(open('plugins/prioritize/.claude-plugin/plugin.json')); print('OK')"
```

Expected: both print `OK` with no errors.

- [ ] **Step 4: Commit**

```bash
git add plugins/prioritize/.claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "feat(prioritize): scaffold plugin and register in marketplace"
```

---

### Task 2: Write the SKILL.md

**Files:**
- Create: `plugins/prioritize/skills/now/SKILL.md`

**Interfaces:**
- Consumes: plugin manifest from Task 1 (resolves skills path to `./skills`)
- Produces: the prompt that drives all skill behavior — MCP check, mode detection, morning kickoff, triage

- [ ] **Step 1: Create SKILL.md**

Create `plugins/prioritize/skills/now/SKILL.md` with this exact content:

````markdown
---
description: Prioritize your work at any point in the day. Runs morning kickoff or mid-day triage automatically based on today's memory graph state.
---

# Work Prioritizer

## Setup (required before first use)

This skill requires the `@modelcontextprotocol/server-memory` MCP server. Add the following to `~/.claude/settings.json` under the `mcpServers` key, then restart Claude Code:

```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"],
  "env": {
    "MEMORY_FILE_PATH": "C:/Users/Rob/Downloads/Journal/memory/priority_graph.json"
  }
}
```

> **Note:** Adjust `MEMORY_FILE_PATH` to match your actual journal path. The `memory/` subdirectory will be created automatically on first use. You only need to do this setup once.

---

## On Every Invocation

**Step 1 — Verify MCP availability**

Call `read_graph`. If the call fails or the tool is not available, stop immediately and display the full Setup section above verbatim. Do not proceed.

**Step 2 — Get today's date**

Determine today's date in `YYYY-MM-DD` format.

**Step 3 — Detect mode**

Call `search_nodes` with the query `session:{today}` (e.g. `session:2026-07-04`).

- No matching entity found → run **Morning Kickoff** below
- Matching entity found → run **Triage** below

No need to tell the user which mode was detected. Just run it.

---

## Morning Kickoff

You are helping the user start their workday with clarity and a concrete priority order.

### 1. Load context

- Attempt to read `~/Downloads/Journal/{today}.txt`. If it exists, extract every unchecked item (`- [ ]`) from the `## Todo` section.
- Attempt to read `~/Downloads/Journal/current_context.txt` for standing project and people context.
- If neither file exists, proceed with an empty task list and rely entirely on conversation.

### 2. Initialize the session

Call `create_entities` with:
```json
[{ "name": "session:{today}", "entityType": "session", "observations": ["started: {ISO timestamp}"] }]
```

### 3. Seed the graph from journal todos

For each unchecked todo item extracted in step 1:

1. Generate a kebab-slug from the task text. Keep it short and meaningful. Examples:
   - "Track outcome of Andrew/Kat Solitaire review" → `track-kat-solitaire-review`
   - "Continue Intusic prototype: instrument representation" → `intusic-instrument-representation`

2. Call `create_entities`:
   ```json
   [{ "name": "task:{slug}", "entityType": "task", "observations": ["status: pending"] }]
   ```

3. If you can identify the relevant project from `current_context.txt` (e.g. Solitaire → `project:solitaire`, Intusic → `project:intusic`, ESG → `project:esg`), call `create_entities` for that project entity if it doesn't already exist, then call `create_relations`:
   ```json
   [{ "from": "task:{slug}", "to": "project:{project-slug}", "relationType": "belongs_to" }]
   ```

### 4. Converse

Ask one question at a time. Wait for the full response before asking the next.

1. *"What's the most important thing you need to finish today?"*
2. *"Anything new, blocking, or competing for your attention?"*

Based on what the user shares, create or update task entities. Assign `priority: 1`, `priority: 2`, `priority: 3` observations to the tasks that matter most. Add `note: {context}` observations for blockers or dependencies the user mentions. Add `blocks` relations where one task explicitly holds up another:
```json
[{ "from": "task:{blocker-slug}", "to": "task:{blocked-slug}", "relationType": "blocks" }]
```

### 5. Output: Today's Top 3

Present this block at the end of the conversation:

```
## Today's Top 3

1. **[Task name]** — [One sentence: why this is #1 given what the user just said]
2. **[Task name]** — [One sentence: why this is #2]
3. **[Task name]** — [One sentence: why this is #3]
```

Keep rationale specific and grounded in what the user told you. No generic productivity advice.

---

## Triage

You are helping the user re-orient after something changed mid-day.

### 1. Load current state

Call `read_graph`. Identify all task entities. Note which have `status: done`, `status: in_progress`, `status: blocked`, or `status: pending`. Note any `blocks` relations between tasks.

### 2. Converse

Ask one question at a time. Wait for the full response before asking the next.

1. *"What just happened — did you finish something, get interrupted, or hit a blocker?"*
2. *"Anything new that came in?"*

### 3. Update the graph

Based on what the user tells you:

- **Task finished:** call `add_observations`:
  ```json
  { "entityName": "task:{slug}", "observations": ["status: done"] }
  ```
- **New task:** call `create_entities` with `status: pending` observation, then `create_relations` for project link (same pattern as Morning Kickoff step 3)
- **Blocker:** call `add_observations` with `note: {blocker description}`, and `create_relations` with `relationType: blocks` if one task is holding up another
- **Reprioritize:** call `add_observations` to update `priority` values on affected tasks

### 4. Output: What to focus on next

Pick the highest-priority non-done, non-blocked task given what just changed. Present:

```
## What to focus on next

**[Task name]**

[2–3 sentences: why this is the right next thing given what the user just told you. Be specific about what changed. No generic advice.]
```

---

## Graph Schema Reference

| Entity | Name pattern | Observations |
|--------|-------------|--------------|
| Session | `session:YYYY-MM-DD` | `started: {timestamp}` |
| Task | `task:{kebab-slug}` | `priority: 1\|2\|3...`, `status: pending\|in_progress\|done\|blocked`, `note: {text}` |
| Project | `project:{kebab-slug}` | — |

| Relation | Meaning |
|----------|---------|
| `task → belongs_to → project` | Task belongs to a project |
| `task → blocks → task` | One task is blocking another |
````

- [ ] **Step 2: Verify file structure**

Run:
```powershell
Get-ChildItem -Recurse plugins/prioritize
```

Expected output includes:
```
plugins/prioritize/.claude-plugin/plugin.json
plugins/prioritize/skills/now/SKILL.md
```

- [ ] **Step 3: Confirm frontmatter is present**

Open `plugins/prioritize/skills/now/SKILL.md` and confirm the file begins with:
```
---
description: Prioritize your work at any point in the day...
---
```

- [ ] **Step 4: Commit**

```bash
git add plugins/prioritize/skills/now/SKILL.md
git commit -m "feat(prioritize): add SKILL.md with morning kickoff and triage flows"
```

---

### Task 3: Write README.md

**Files:**
- Create: `plugins/prioritize/README.md`

**Interfaces:**
- Consumes: nothing from prior tasks beyond knowing the plugin name and skill invocation
- Produces: user-facing setup and usage docs

- [ ] **Step 1: Create README.md**

Create `plugins/prioritize/README.md` with this exact content:

```markdown
# prioritize

Work prioritizer for Claude Code. Uses a local knowledge graph (via MCP) to run a morning kickoff or mid-day triage session — determined automatically from today's graph state.

## Setup

**1. Configure the MCP server**

Add the following to `~/.claude/settings.json` under the `mcpServers` key:

```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"],
  "env": {
    "MEMORY_FILE_PATH": "C:/Users/Rob/Downloads/Journal/memory/priority_graph.json"
  }
}
```

> **Important:** Change `MEMORY_FILE_PATH` to match your actual journal directory. The `memory/` subdirectory will be created automatically. This step only needs to be done once.

**2. Restart Claude Code**

The MCP server is started by Claude Code on launch — a restart is required after changing `settings.json`.

**3. Install the plugin**

```
/plugin install prioritize --local ./plugins/prioritize
```

## Usage

```
/prioritize:now
```

**First invocation of the day** → Morning kickoff mode:
- Reads today's journal (`YYYY-MM-DD.txt`) and extracts unchecked todos
- Reads `current_context.txt` for project context
- Asks two questions, then outputs your top 3 priorities with rationale

**Any subsequent invocation** → Triage mode:
- Reads today's tasks from the graph
- Asks what changed
- Updates the graph and tells you what to focus on next

## Works best with the journal skill

Run `/journal:journal` at the end of your day. The `/prioritize:now` morning kickoff reads that entry's todo list automatically, so your priorities start from your own notes rather than from scratch.
```

- [ ] **Step 2: Commit**

```bash
git add plugins/prioritize/README.md
git commit -m "feat(prioritize): add README with setup and usage instructions"
```

---

### Task 4: Local install and smoke test

**Files:** None created. Verification only.

**Interfaces:**
- Consumes: all files from Tasks 1–3

- [ ] **Step 1: Configure MCP in settings.json**

Open `~/.claude/settings.json`. If `mcpServers` key doesn't exist, add it. Add the `memory` entry:

```json
"mcpServers": {
  "memory": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-memory"],
    "env": {
      "MEMORY_FILE_PATH": "C:/Users/Rob/Downloads/Journal/memory/priority_graph.json"
    }
  }
}
```

Save the file and restart Claude Code.

- [ ] **Step 2: Install the plugin locally**

In Claude Code, run:
```
/plugin install prioritize --local ./plugins/prioritize
```

Expected: success message, plugin listed as installed.

- [ ] **Step 3: Verify skill appears**

Run `/` and check that `prioritize:now` appears in the skill picker with description "Prioritize your work at any point in the day."

- [ ] **Step 4: Smoke test morning kickoff**

Invoke `/prioritize:now`. Verify:
- Skill reads the graph (no MCP error)
- No `session:{today}` found → morning mode runs
- Skill reads `current_context.txt` and/or today's journal
- Asks "What's the most important thing you need to finish today?"
- After answering both questions, outputs a **Today's Top 3** block

- [ ] **Step 5: Smoke test triage**

Without restarting, invoke `/prioritize:now` again. Verify:
- `session:{today}` is now found → triage mode runs
- Asks "What just happened..."
- After answering both questions, outputs a **What to focus on next** block

- [ ] **Step 6: Final commit if any fixups were needed**

If the smoke test revealed prompt issues that required edits to SKILL.md, commit those:
```bash
git add plugins/prioritize/skills/now/SKILL.md
git commit -m "fix(prioritize): adjust prompt based on smoke test"
```
