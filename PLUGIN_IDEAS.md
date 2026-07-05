# Plugin Ideas

Organized by what they consume and what gap they fill. Artifact chains are noted explicitly.

---

## Tier 1 — Directly consume existing plugin artifacts

### standup
**Consumes:** yesterday's `YYYY-MM-DD.md` (journal) + today's `priority_graph.json` (prioritize)
**Produces:** a standup message (text or saved `standup-YYYY-MM-DD.md`)

Reads what you captured last night and what's on the graph this morning, then drafts a standup in your voice. Asks one clarifying question ("anything blocked?") then outputs a 3-part standup: done / doing / blockers. Optionally posts to Slack via MCP.

Gap filled: the journal and priority graph already contain everything needed for a standup — right now you have to reconstruct it manually.

---

### weekly-digest
**Consumes:** all `YYYY-MM-DD.md` entries for the past 7 days (journal) + `priority_graph.json` (prioritize)
**Produces:** `weekly-digest-YYYY-WW.md`

Summarizes the week: what shipped, what stalled, recurring themes, and a "headline" for the week. Different from introspection — this is a document artifact, not a conversation. Useful for async updates to teammates or stakeholders.

Gap filled: introspection surfaces patterns conversationally but produces nothing you can send or reference later.

---

### action-items
**Consumes:** all `YYYY-MM-DD.md` entries (journal) + any `usability-report-*.md` files in the project
**Produces:** `action-items.md` — a deduplicated, consolidated todo list with source references

Scans journal entries for unchecked `- [ ]` items and usability report recommendations, deduplicates, and groups by theme. Each item links back to the source file and date.

Gap filled: todos accumulate across journal entries and usability reports with no single view. Prioritize works from the graph but doesn't aggregate file-based todos.

---

### feature-spec
**Consumes:** a research session's synthesis output + a `usability-report-*.md` file
**Produces:** `spec-[feature-name]-YYYY-MM-DD.md`

Bridges research and UX findings into a structured feature spec: problem statement, user needs (from usability findings), approach options (from research), open questions. Asks a few targeted questions to fill gaps, then writes the doc.

Gap filled: research and usability-report both produce inputs that naturally lead to a spec, but nothing connects them.

---

### context-snapshot
**Consumes:** recent journal entries + `priority_graph.json` + any open usability reports or research notes
**Produces:** `current_context.txt` — the file that `prioritize:now` reads on morning kickoff

Rebuilds or refreshes the context file that the prioritize plugin reads. Run it when context has drifted or after a long break. Effectively the "connective tissue" between all other plugins.

Gap filled: `current_context.txt` is currently written manually. Nothing generates it from the other artifacts.

---

## Tier 2 — Extend an existing plugin

### decision-log
**Complements:** journal (captures decisions separately from narrative)
**Produces:** `decisions/YYYY-MM-DD-[slug].md`

When you're weighing a choice, interviews you about the options, constraints, and tradeoffs — then writes a structured decision record (context / options / decision / rationale). The journal captures what happened; this captures why.

Introspection can then surface decision patterns ("you tend to defer infrastructure decisions").

---

### retrospective
**Consumes:** week's journal entries + priority graph history
**Produces:** `retro-YYYY-WW.md`

Structured sprint retrospective: what went well, what didn't, what to change. Seeds each section from the journal and graph rather than asking from scratch. Asks follow-up questions only where the data is thin.

Different from weekly-digest: this is future-facing (what to change) not just a summary.

---

### ux-ticket
**Consumes:** a `usability-report-*.md` file
**Produces:** structured GitHub issues or Linear tickets (one per severity 3–4 finding)

Reads a usability report and converts each high-severity finding into a well-formed bug/feature ticket with: title, description, steps to reproduce (if UI), severity, heuristic violated. Optionally creates them via GitHub MCP.

Gap filled: usability-report surfaces findings but leaves ticket-creation as manual work.

---

### research-note
**Extends:** research (gives it a saved artifact)
**Produces:** `research/[topic]-YYYY-MM-DD.md`

Wraps the research plugin's final synthesis into a saved Markdown note with: topic, date, key findings, open questions, and recommended next steps. The note can then be consumed by feature-spec or journal.

Gap filled: research currently produces no persistent artifact — all findings live in conversation context only.

---

## Tier 3 — Fill workflow gaps

### daily-brief
**Consumes:** `priority_graph.json` + yesterday's journal + Google Calendar MCP (today's events)
**Produces:** a morning briefing in conversation

Combines your priorities, yesterday's capture, and today's calendar into a single morning brief before you touch prioritize. Surfaces conflicts ("you have 3 hours of meetings and 4 priorities — what gives?").

---

### goal-tracker
**Consumes:** introspection patterns over time + journal entries
**Produces:** `goals/[quarter].md` — quarterly goals with check-in notes

Structured goal-setting that persists across the quarter. Introspection can reference it ("you said your Q3 goal was X — journal entries suggest you've been avoiding it"). Run it at the start of a quarter; check in monthly.

---

### learning-log
**Complements:** journal (journal captures work; this captures learning)
**Produces:** `learning/YYYY-MM-DD.md`

Like journal but focused on what you studied or learned: topic, source, key insight, what you want to explore next. Feeds into tech-radar and introspection. Useful for tracking deliberate skill growth separate from day-to-day work.

---

### tech-radar
**Consumes:** research notes + learning-log entries + journal mentions of tools/libraries
**Produces:** `tech-radar.md` — a living document in quadrants (adopt / trial / assess / hold)

Tracks technologies you've encountered across all your plugin artifacts and helps you maintain a personal stance on each. Useful for consulting/contracting work where you need to stay current across multiple stacks.

---

## Artifact map

```
journal ──────────────────────────────────────────────────────┐
  └─ YYYY-MM-DD.md ─→ standup, weekly-digest, action-items,  │
                       retrospective, context-snapshot         │
                                                               │
prioritize ────────────────────────────────────────────────────┤
  └─ priority_graph.json ─→ standup, weekly-digest,           │
                             retrospective, context-snapshot   │
                                                               │
usability-report ──────────────────────────────────────────────┤
  └─ usability-report-*.md ─→ action-items, feature-spec,     │
                               ux-ticket, context-snapshot     │
                                                               │
research ───────────────────────────────────────────────────── ┤
  └─ (conversation only) ─→ research-note ─→ feature-spec     │
                                                               │
introspection ─────────────────────────────────────────────────┘
  └─ (conversation only) ─→ goal-tracker, learning-log
```
