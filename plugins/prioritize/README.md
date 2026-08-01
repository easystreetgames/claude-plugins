# prioritize

Work prioritizer for Claude Code. Uses a local knowledge graph (via MCP) to run a morning kickoff or mid-day triage session — determined automatically from today's graph state.

## Setup

**1. Configure the MCP server**

Create or edit `~/.claude/.mcp.json` (your global MCP config) and add the `memory` entry:

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"],
      "env": {
        "MEMORY_FILE_PATH": "C:/Users/Rob/Downloads/Journal/memory/priority_graph.json"
      }
    }
  }
}
```

> **Important:** Change `MEMORY_FILE_PATH` to match your actual journal directory. The `memory/` subdirectory will be created automatically. This step only needs to be done once.

**2. Restart Claude Code**

The MCP server is started by Claude Code on launch — a restart is required after adding the `.mcp.json` entry.

**3. Install the plugin**

```
/plugin install prioritize --local ./plugins/prioritize
```

## Usage

```
/prioritize:now
```

**First invocation of the day** → Morning kickoff mode:
- Reads today's journal (`YYYY-MM-DD.md`) and extracts unchecked todos
- Reads `current_context.md` for project context
- Asks two questions, then outputs your top 3 priorities with rationale

**Any subsequent invocation** → Triage mode:
- Reads today's open tasks from the graph
- Asks what changed
- Updates the graph (lean status/priority only) and tells you what to focus on next
- Blocker detail is appended to `log.md`, not stored as free-form graph notes

## Lean memory graph

The MCP graph is an **index**, not a notebook:

- Holds open tasks (status + priority), projects, today's session, pending 1:1 topics
- Prunes old sessions and done tasks on each invoke
- Narrative (blockers, context) goes in `{journal-path}/log.md` and daily journal files

If an existing `priority_graph.json` has grown large from older skill versions, delete it (or hand-edit) once; skills will rebuild open state from journals on the next run.

## Works best with the journal skill

Run `/journal:journal` at the end of your day. The `/prioritize:now` morning kickoff reads that entry's todo list automatically, so your priorities start from your own notes rather than from scratch.

