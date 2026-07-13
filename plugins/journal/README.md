# Journal Plugin

A conversational journaling assistant for Claude Code. It interviews you about your day, searches for relevant tech news, and writes a structured journal entry with todo list to `~/Downloads/Journal/YYYY-MM-DD.txt`.

## Install

```
/plugin marketplace add <marketplace-url>
/plugin install journal@esgames-plugins
```

## Skills

### `/journal:journal`

Starts a journaling session. Claude will:

1. Load any pending log entries (see `/journal:log`) as conversation context
2. Ask open-ended questions about your day and follow up until you say **"that's all"**
3. Search for relevant tech news to include in a "Tech Context" section
4. Write a structured entry (activities, tech context, todo list) to `~/Downloads/Journal/YYYY-MM-DD.txt`
5. Update `~/Downloads/Journal/current_context.txt` for the next session
6. Clear the log file so entries aren't replayed

### `/journal:log <note>`

Appends a quick timestamped note to `~/Downloads/Journal/log.txt` for pickup during the next journal session. Useful for capturing things mid-day without starting a full session.

```
/journal:log fixed the tangle game bug
/journal:log shipped new onboarding flow
```

Natural language also works:

```
Log that I added a new tangle game
```

Entries are consumed and cleared automatically when `/journal:journal` runs.

## Output

Entries are saved to:

```
~/Downloads/Journal/YYYY-MM-DD.txt
```

Each entry includes a summary of activities, a Tech Context section with relevant news links, and a Todo checklist.
