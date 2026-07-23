---
description: Set up or view the local journal folder for this project. Run once per project to configure where journal files are stored.
---

# Journal Path

You manage the journal folder location for the current project. All other journal skills read `.claude/journal-path.txt` to discover where journal files live.

## What to do

1. **Read the current config**: Check whether `.claude/journal-path.txt` exists in the current working directory.

2. **Determine the path**:
   - If the user passed an argument (e.g. `/journal:path C:\Users\Rob\Documents\Journal`), use that as the path.
   - If the config file exists and no argument was given, read and display the current path — then ask if they want to change it.
   - If neither, default to `./journal` relative to the current working directory.

3. **Write the config**: Write the chosen path to `.claude/journal-path.txt`. Create the `.claude/` directory if needed.

4. **Create the journal folder**: Create the directory at the chosen path if it doesn't exist. Also create a `research/` subdirectory inside it.

5. **Confirm**: Tell the user the journal folder path and whether it was created or already existed.

## Notes

- All journal skills read `.claude/journal-path.txt` automatically before any file operations.
- To change the path later: run `/journal:path <new-path>`.
- The `.claude/` folder is the standard location for Claude Code project configuration.
- If you use the `prioritize` skill's MCP memory server, update `MEMORY_FILE_PATH` in your MCP config to point to `{journal-path}/memory/priority_graph.json`.
