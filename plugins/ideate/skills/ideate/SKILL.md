---
name: ideate
description: Use when asked to brainstorm, generate ideas, or get a ranked idea list on a topic — dispatches subagents to generate ideas, invents three diverse personas for the topic, has each score the ideas independently, then presents the list sorted by normalized score
---

# Ideate

## Overview

Topic in, ranked idea list out. Every phase runs in a subagent; the main context only coordinates, normalizes scores, saves the full reviews to a local Markdown file, and presents results.

**Topic**: `$ARGUMENTS`. If empty, ask the user for a topic (and optionally how many ideas — default 12) before doing anything else.

## Workflow

**Phase 1 — Ideas and personas (parallel)**

Dispatch these two subagents in the same message so they run in parallel. They must not see each other's output — personas are designed without knowing the list, so they can't be tailored to it.

*Idea generator:*

> Generate [N] distinct ideas for: [TOPIC]. Cover a wide range — obvious and safe, ambitious, unconventional, cheap/quick wins. No near-duplicates. Return a numbered list; each item is a short title (≤ 8 words) followed by a one-sentence description. Return only the list.

*Persona designer:*

> Design 3 personas who would evaluate ideas about: [TOPIC]. Make them genuinely diverse — different roles, incentives, and risk tolerance (e.g., a practitioner, a skeptic or budget-holder, an end user or outsider). For each return: name, role, one sentence on what they care about most, and one sentence on what makes them reject an idea. Return only the 3 personas as a numbered list.

---

**Phase 2 — Persona reviews (parallel)**

Dispatch 3 subagents in one message, one per persona. Each gets the full idea list (with original numbers) and only its own persona:

> You are [NAME], [ROLE]. You care most about: [VALUES]. You reject ideas that: [DEALBREAKER].
>
> Score every idea below from 1 to 10 from your perspective. Use the full range — your best idea should be near 10 and your worst near 1. Do not give every idea the same score.
>
> [NUMBERED IDEA LIST]
>
> Return exactly one line per idea, in this format and nothing else:
> `<idea number> | <score> | <one-sentence reason in your voice>`

If a reviewer's output is missing ideas or malformed, re-dispatch that reviewer once with the same prompt.

---

**Phase 3 — Normalize and rank (main context)**

Raw scores aren't comparable across personas — one may grade harshly, another generously. Normalize each persona's scores before combining:

1. For each persona, min-max scale their scores to 0–10: `norm = (raw − min) / (max − min) × 10`. If a persona gave every idea the same score, set all their normalized scores to 5.
2. Each idea's **final score** = mean of its 3 normalized scores, rounded to one decimal.
3. Sort descending by final score. Break ties by the higher minimum normalized score (the idea nobody hated wins).

---

**Phase 4 — Write the full reviews file (main context)**

Before presenting, save every review to a local Markdown file so the ranking can be audited later. Path: `ideate/YYYY-MM-DD-<topic-slug>.md` under the current working directory (create the folder if needed; if the file exists, append `-2`, `-3`, …). Use this structure:

```markdown
# Ideate: <topic>

Date: YYYY-MM-DD

## Personas

### 1. <Name> — <role>
- **Cares about:** <values>
- **Rejects:** <dealbreaker>

(repeat for all 3)

## Ideas

1. **Title** — description
(full numbered list, original numbers)

## Reviews

### <Persona 1 name>

Raw range: <min>–<max>

| # | Idea | Raw | Norm | Reason |
|---|------|-----|------|--------|
| 1 | Title | 7 | 6.7 | <their reason, verbatim> |

(one table per persona, every idea, in idea-number order)

## Final ranking

(the same ranked table shown in Phase 5)
```

Reasons are copied verbatim from the reviewer output — don't paraphrase or trim them.

---

**Phase 5 — Present**

Show the personas first (one line each: name — role — what they care about), then the ranked table:

```markdown
| Rank | Idea | Score | <Persona 1> | <Persona 2> | <Persona 3> |
|------|------|-------|-------------|-------------|-------------|
| 1 | **Title** — description | 8.7 | 8 → 9.0 | 10 → 10.0 | 6 → 7.1 |
```

Persona columns are **required** — never collapse the output to the final score alone. Each cell shows the persona's raw score → normalized score. After the table, add:

- **Top pick notes** — for the top 3 ideas, each persona's one-line reason.
- **Most divisive** — the idea with the largest spread between its highest and lowest normalized score, with the two opposing reasons.
- **Full reviews** — the path to the file from Phase 4, noting it has every persona's reason for every idea.

Then ask whether they want to regenerate with a different angle, swap a persona, or expand on any idea.

## Rules

- **Never generate ideas or scores in the main context.** All content comes from subagents; the main context only does arithmetic and formatting.
- **Reviewers are independent.** Never show one persona's scores to another.
- **Keep idea numbers stable** across phases so scores map back correctly.
- **Show your normalization honestly.** Don't adjust scores by hand to make the ranking look better.

## Subagent Tools

Subagents need no tools by default. Give the idea generator `WebSearch` only if the topic depends on current events or market facts.
