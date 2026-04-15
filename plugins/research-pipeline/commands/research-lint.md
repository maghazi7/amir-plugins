---
name: research-lint
description: Scan the research wiki for orphans, missing entities, broken wikilinks, empty overviews, unprocessed raw, contradictions, stale claims, duplicate entities, and data gaps
argument-hint: [--scope=<topic|all>] [--auto-fix]
---

# /research-lint Command

Runs the research wiki health check. Produces a dated report at `Research/_lint-reports/YYYY-MM-DD-lint.md`.

## Argument parsing

Arguments are in `$ARGUMENTS`. Parse:

- **`--scope=<topic-slug>`** — limit to a single topic folder (default: all of `Research/`)
- **`--scope=all`** — explicit full scan (default behavior)
- **`--auto-fix`** — apply safe auto-fixes (see `references/lint-checks.md` for which checks auto-fix). Default: off. Manual review recommended.

## Execution

Invoke the `research-pipeline` skill in lint mode. The skill follows `references/lint-checks.md` exactly:

1. Scan the wiki (scope-limited per argument)
2. Run all 9 checks
3. Apply safe auto-fixes if `--auto-fix` is set
4. Write the dated report to `Research/_lint-reports/YYYY-MM-DD-lint.md`
5. Append a lint entry to `Research/_log.md`
6. Present the summary to the user

## Checks (summary)

| # | Check | Auto-fix? |
|---|---|---|
| 1 | Orphans — entity/synthesis files with no inbound wikilinks | No |
| 2 | Missing entity pages — names mentioned in synthesis, no entity page | Yes (seed from first mention) |
| 3 | Broken wikilinks — `[[target]]` that doesn't resolve | Yes (if target exists at different slug) |
| 4 | Empty overviews — topic has sources but `overview.md` says "No facts yet" | Yes (re-run compounding) |
| 5 | Unprocessed raw — `_raw/` file with empty `synthesis_targets` | Yes (offer to re-run compounding) |
| 6 | Contradictions (LLM-judged) — conflicting facts | No (user picks) |
| 7 | Stale claims — entity with `last_updated` > 30 days + newer raw mentioning it | No (report with suggestion) |
| 8 | Duplicate entities — pages that look like aliases of the same thing | No (report with merge suggestion) |
| 9 | Data gaps / suggestions — entities with source_count 1, thin overviews, unanswered open questions | No (report only) |

## Summary output

```
Lint complete.
  Report: [[_lint-reports/YYYY-MM-DD-lint]]
  Orphans: N
  Missing entity pages: N (auto-fixed: M)
  Broken wikilinks: N (auto-fixed: M)
  Empty overviews: N (auto-fixed: M)
  Unprocessed raw: N
  Contradictions: N
  Stale claims: N
  Duplicate entities: N
  Data gaps: N
```

## Triggers

- Manual: `/research-lint` any time
- Auto: at the end of every `/research-batch`
- Scheduled (optional): weekly via the `/loop` skill — off by default
