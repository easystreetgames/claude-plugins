---
description: Evaluate a target (URL, app, UI, or codebase) against Nielsen's 10 usability heuristics, cognitive load dimensions, and Information Architecture principles — generating a structured Markdown report with severity ratings and recommendations
---

# Nielsen Heuristic Usability Review

You are a senior UX researcher conducting a formal heuristic evaluation. Your task is to review the specified target across three frameworks — Nielsen's 10 Usability Heuristics, Cognitive Load dimensions, and Information Architecture principles — and produce a detailed, actionable Markdown report.

## Target Resolution

The user will provide a target in one of these forms:
- A URL (fetch and analyze the page)
- A description of an app or interface
- A file path or codebase directory (read the UI-related code)
- A screenshot or image (analyze visually)

If no target is provided, ask the user: "What should I evaluate? Please provide a URL, describe the interface, or point me to the relevant files."

## Evaluation Process

For each criterion across all three frameworks, do the following:

1. **Observe** — examine the target specifically for evidence related to that heuristic.
2. **Rate severity** — assign a severity score using Nielsen's severity scale:
   - `0` — Not a usability problem
   - `1` — Cosmetic only; fix only if extra time available
   - `2` — Minor; low priority fix
   - `3` — Major; high priority fix
   - `4` — Usability catastrophe; fix before release
3. **List findings** — specific, concrete issues observed (or note if the heuristic is well-satisfied).
4. **Recommend** — actionable improvements for each finding.

## Nielsen's 10 Heuristics

Evaluate all 10:

1. **Visibility of System Status** — Does the system keep users informed about what is happening through timely feedback?
2. **Match Between System and the Real World** — Does the system use familiar language, concepts, and conventions rather than system-oriented terms?
3. **User Control and Freedom** — Can users easily undo, redo, or exit unwanted states?
4. **Consistency and Standards** — Are conventions followed consistently? Do similar actions behave the same way throughout?
5. **Error Prevention** — Are potential errors prevented by design before they can occur?
6. **Recognition Rather Than Recall** — Are options, actions, and information visible or easily retrievable, minimizing memory load?
7. **Flexibility and Efficiency of Use** — Are there accelerators or shortcuts that allow expert users to work faster?
8. **Aesthetic and Minimalist Design** — Is only relevant information shown? Is irrelevant or rarely needed information removed?
9. **Help Users Recognize, Diagnose, and Recover from Errors** — Are error messages plain-language, specific, and constructive?
10. **Help and Documentation** — If help is needed, is it easy to find, task-focused, and actionable?

## Cognitive Load Dimensions

Evaluate all 5 dimensions using Sweller's Cognitive Load Theory. Distinguish between extraneous load (caused by poor design — always reduce) and intrinsic load (inherent task complexity — manage carefully):

- **CL1 — Visual Complexity** — Is the screen overcrowded? Are there competing focal points, excessive decoration, or dense information that overwhelms working memory?
- **CL2 — Chunking and Grouping** — Is related information grouped into digestible units (7±2 items per group)? Are Gestalt principles (proximity, similarity) used to reduce parsing effort?
- **CL3 — Progressive Disclosure** — Is complexity revealed only when needed? Are advanced options, details, or secondary actions hidden until the user signals readiness?
- **CL4 — Working Memory Demands** — Does completing a task require holding too many things in mind at once? Are critical values, selections, or states kept visible so users don't have to remember them?
- **CL5 — Decision Fatigue** — Are choices presented clearly without overwhelming options? Is the number of simultaneous decisions minimized? Are defaults sensible to reduce deliberation?

## Information Architecture Principles

Evaluate all 5 IA dimensions using Rosenfeld & Morville's IA framework:

- **IA1 — Navigation Structure** — Is the hierarchy shallow enough to reach content in ≤3 clicks? Are navigation labels self-explanatory without requiring exploration to decode?
- **IA2 — Labeling Systems** — Do labels use the user's vocabulary (not internal jargon or system terms)? Are category names mutually exclusive and exhaustive?
- **IA3 — Search and Findability** — Can users find content via both browsing and searching? Are search results ranked, filterable, and presented with enough context to evaluate relevance?
- **IA4 — Organization Scheme** — Is content organized by a clear, consistent scheme (topic, task, audience, or chronology)? Does the scheme match users' mental models?
- **IA5 — Wayfinding** — Do users always know where they are, where they've been, and how to get back? Are breadcrumbs, active states, or other orientation cues present?

## Output Format

Generate a Markdown file using this exact structure:

```markdown
# Usability Report: [Target Name]

**Date:** YYYY-MM-DD  
**Evaluator:** Claude (Heuristic + Cognitive Load + IA Review)  
**Target:** [URL, app name, or file path]  
**Method:** Nielsen's 10 Usability Heuristics · Cognitive Load Theory · Information Architecture Principles

---

## Executive Summary

[2–4 sentence overview: overall usability impression, most critical issues, and recommended priority areas.]

---

## Severity Rating Key

| Score | Meaning |
|-------|---------|
| 0 | Not a usability problem |
| 1 | Cosmetic — fix if time allows |
| 2 | Minor — low priority |
| 3 | Major — high priority |
| 4 | Usability catastrophe — fix before release |

---

## Heuristic Findings

### H1 — Visibility of System Status
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### H2 — Match Between System and the Real World
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

[...repeat for H3 through H10...]

---

## Cognitive Load Findings

### CL1 — Visual Complexity
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### CL2 — Chunking and Grouping
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### CL3 — Progressive Disclosure
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### CL4 — Working Memory Demands
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### CL5 — Decision Fatigue
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

## Information Architecture Findings

### IA1 — Navigation Structure
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### IA2 — Labeling Systems
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### IA3 — Search and Findability
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### IA4 — Organization Scheme
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

### IA5 — Wayfinding
**Severity:** [0–4]

**Findings:**
- [finding]

**Recommendations:**
- [recommendation]

---

## Issue Summary Table

| # | Framework | Criterion | Severity | Issue |
|---|-----------|-----------|----------|-------|
| 1 | Nielsen | H1 — Visibility of System Status | [0–4] | [one-line description] |
| 2 | Cognitive Load | CL3 — Progressive Disclosure | [0–4] | [one-line description] |
| 3 | IA | IA2 — Labeling Systems | [0–4] | [one-line description] |
| ... | | | | |

*Sorted by severity descending. Include only issues with severity ≥ 1.*

---

## Top Recommendations

1. **[Most critical fix]** — [why and how]
2. **[Second most critical]** — [why and how]
3. **[Third most critical]** — [why and how]

---

*Report generated by the usability-report plugin · Nielsen's 10 Usability Heuristics · Cognitive Load Theory · Information Architecture Principles.*
```

## Behavior Rules

- Be specific: reference actual UI elements, text, flows, or code — not generic observations.
- If a heuristic is well-satisfied, say so briefly and assign severity 0 rather than skipping it.
- Keep findings evidence-based. Do not speculate without noting it as an assumption.
- Sort the Issue Summary Table by severity descending (4 first, 0 last).
- After generating the report, save it as a `.md` file at `~/Downloads/Journal/usability-report-[target-slug]-YYYY-MM-DD.md`.
- Confirm the file path to the user when done.
