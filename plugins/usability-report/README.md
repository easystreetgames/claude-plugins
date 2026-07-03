# usability-report

A Claude Code plugin that evaluates a target against [Nielsen's 10 Usability Heuristics](https://www.nngroup.com/articles/ten-usability-heuristics/) and generates a structured Markdown report with severity ratings and recommendations.

## Install

```
/plugin install usability-report@esgames-plugins
```

## Skills

### `/usability-report:nielsen-review`

Runs a full heuristic evaluation and saves a Markdown report to your working directory.

**Usage:**

```
/usability-report:nielsen-review https://example.com
/usability-report:nielsen-review src/components/
/usability-report:nielsen-review Describe the checkout flow of my app
```

**What it produces:**

- Executive summary
- Per-heuristic findings with severity scores (0–4)
- Concrete, actionable recommendations
- Issue summary table sorted by severity
- Top priority fix list
- Saved as `usability-report-[target]-YYYY-MM-DD.md`

## Severity Scale

| Score | Meaning |
|-------|---------|
| 0 | Not a usability problem |
| 1 | Cosmetic — fix if time allows |
| 2 | Minor — low priority |
| 3 | Major — high priority |
| 4 | Usability catastrophe — fix before release |

## Nielsen's 10 Heuristics Covered

1. Visibility of System Status
2. Match Between System and the Real World
3. User Control and Freedom
4. Consistency and Standards
5. Error Prevention
6. Recognition Rather Than Recall
7. Flexibility and Efficiency of Use
8. Aesthetic and Minimalist Design
9. Help Users Recognize, Diagnose, and Recover from Errors
10. Help and Documentation
