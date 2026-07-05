---
name: research
description: Use when asked to research a topic, investigate an unfamiliar domain, or explore a question in depth — dispatches subagents so the main context stays clean and the user can steer focus iteratively
---

# Research

## Overview

Iterative research using subagents: broad overview first, then user-directed deep dives. Each phase gets its own subagent — the main context only holds findings and conversation.

## Workflow

**Phase 1 — Broad sweep**

Dispatch a subagent with this goal:

> Research [TOPIC] broadly. Return structured markdown with 5–8 major aspects/subtopics, each as a numbered item with: a 2–3 sentence summary, 2–3 key terms or names, and one sentence on why it matters. Be concise and scannable. Cite sources where possible.

Present the numbered list to the user verbatim. Then ask:

> Which aspect would you like to explore deeper? Pick a number, name a few, or say "done" if this covers it.

---

**Phase 2 — Deep dive**

For each user-selected aspect, dispatch a focused subagent:

> Deep dive on [ASPECT] in the context of [ORIGINAL TOPIC]. Cover: how it works, key examples or players, current state, open questions or controversies, and practical implications. 500–800 words, structured markdown with headers. Cite sources.

Present the findings, then ask:

> Want to go deeper on anything from this, explore another aspect, or are you done?

Repeat Phase 2 as many times as the user wants.

---

**Phase 3 — Close**

When the user says they're done, offer a brief synthesis (3–5 bullets) connecting the aspects they explored, then stop.

## Rules

- **Never research in the main context.** Always delegate to a subagent.
- **Always present findings before asking for direction.** Don't ask what they want until they've seen results.
- **Number every aspect in the broad sweep** so the user can steer by number.
- **One subagent per deep dive.** Don't batch multiple topics into one subagent — focus produces better results.
- **Match depth to the question.** A broad topic needs 7–8 aspects; a narrow one needs 4–5. Don't pad.

## Subagent Tools

Give subagents: `WebSearch`, `WebFetch`. Add `Read` if the research involves local files or a codebase.

## Example Flow

```
User: "Research large language model training"

→ Broad subagent → 7 aspects returned:
  1. Pre-training (next-token prediction at scale)
  2. Fine-tuning and RLHF
  3. Compute and infrastructure
  4. Datasets and data quality
  5. Emergent capabilities
  6. Alignment and safety
  7. Inference optimization

Presented to user. User: "Tell me more about #4 and #6"

→ Deep dive subagent on "Datasets and data quality" → findings presented
→ Deep dive subagent on "Alignment and safety" → findings presented

User: "Go deeper on RLHF from #2"

→ Deep dive subagent on "RLHF in LLM training" → findings presented

User: "Done"

→ Synthesis: 3–5 bullets connecting data quality → training quality → alignment needs
```
