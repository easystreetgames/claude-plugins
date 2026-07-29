---
description: Sanitize a standup report for team posting — strip sensitive details, clarify jargon, remove times, keep only what teammates need
---

# Standup Sanitize

You are preparing a standup report for posting to a team channel. Take the standup report in the current conversation (from `/standup:standup` output or pasted by the user) and produce a clean, team-ready version.

## What to do

**1. Identify the source report**

Look for a standup in the current conversation that follows the Yesterday / Today / Blockers format. If none is present, ask the user to paste one or run `/standup:standup` first.

**2. Apply these transformations**

- **Remove times and timestamps.** Drop phrases like "at 9am", "by EOD", "this afternoon", "3pm sync" — the team doesn't need scheduling details.
- **Strip internal-only details.** Remove references to internal tools, personal systems, or in-flight context that has no meaning to teammates (e.g., "updated my local journal", "ran `/prioritize:now`", "tested the dev build on my machine").
- **Clarify jargon and abbreviations.** Expand or rephrase acronyms and shorthand that teammates outside your immediate context may not recognize (e.g., "FTUE" → "first-time user experience", "ESG" → use the full project name if it would be unclear).
- **Rephrase anything that could reveal sensitive information.** If a bullet references HR matters, personnel issues, compensation, or anything that should not be in a team channel, remove or generalize it (e.g., "discussed comp with manager" → omit entirely).
- **Keep it team-relevant.** Every bullet that survives should answer: "Does my team need to know this to coordinate with me?" If no, drop it.

**3. Preserve structure**

Output the sanitized report in exactly the same format:

```
**Yesterday**
- [bullet]

**Today**
- [bullet]

**Blockers**
- [bullet]
```

Omit the Blockers section if there are none. Do not add preamble or trailing commentary.

**4. Offer a diff**

After the sanitized report, show a brief summary of what was changed or removed — one line per significant edit. Example:

> Removed: time references (3pm, EOD)
> Removed: internal tool reference (journal skill)
> Clarified: "FTUE" → "first-time user experience"

Then ask: *"Ready to post, or any adjustments?"*
