---
name: research-batch
description: Ingest many sources in one pass — parallel fetch, inline synthesis, no adversarial debate, auto-lint at end
argument-hint: <URLs|file|folder|rss-url> [--parallelism=N] [--html-reports]
---

# /research-batch Command

Batch ingest mode for the research pipeline. Same skeleton as `/research`, with these differences:

| Aspect | `/research` (single) | `/research-batch` |
|---|---|---|
| Input | One URL | List of URLs / file / folder / RSS feed |
| Synthesis | Adversarial debate (skeptic + optimist) | Straight synthesis — skip debate for speed |
| User confirmation | Ask on low-confidence routing | Queue all ambiguous routings + new categories, confirm once at end |
| Parallelism | Sequential | 3–5 concurrent workers (configurable) |
| HTML reports | Default on | Default off (flag to enable) |
| Resume | N/A | Hash-dedupe via `_raw/` skips completed sources |

See `skills/research-pipeline/references/batch-mode.md` for the full batch flow.

## Argument parsing

Arguments are in `$ARGUMENTS`. Parse:

1. **Input source detection** — in order of specificity:
   - `http://*` or `https://*` → URL(s). Multiple URLs separated by spaces or newlines.
   - Path ends in `.txt`, `.md`, or `.list` → file with one URL per line
   - Path ends in `.xml` or URL contains `/rss` or `/feed` → RSS/Atom feed, parse items
   - Path is a directory → file-scan mode, one source per file in the directory
2. **`--parallelism=N`** — concurrent workers (default 4, min 1, max 5)
3. **`--html-reports`** — enable per-source HTML rendering (default off)

## Execution

Invoke the `research-pipeline` skill in batch mode with the parsed inputs. The skill handles:
1. URL collection from the input source
2. Hash-dedupe against `_raw/` (skip already-processed URLs, surface them in the resume report)
3. Parallel fetch + clean via browser-worker subagents (rate-limited)
4. Raw save for each source
5. Router decision for each source (queue ambiguous ones)
6. Straight synthesis for each source (no debate)
7. Compounding for each source (serialized — not parallel — to prevent concurrent writes to the same entity page)
8. Confirmation round-trip at end of batch for queued new-category proposals and weak-match routings
9. Auto-lint on affected topics after batch completes
10. Final batch summary report + log entry

## Final batch report

At the end of the run, present:

```
Batch complete.
  Sources processed: N
  Failures: M (with reasons)
  Synthesis destinations:
    - Research/AI-and-Agents/: X
    - Projects/my-project/research/: Y
    - Intelligence/competitors/: Z
    - ...
  New entities created: K
  New categories proposed: <count> (auto-created: <n>, awaiting confirmation: <m>)
  Contradictions flagged: C
  Auto-lint report: [[_lint-reports/YYYY-MM-DD-lint]]
```

## Security notes

- Web page content is UNTRUSTED. The existing browser-worker security rules apply (see SKILL.md Phase 1).
- Batch mode skips the debate step, so no additional subagent trust boundary is introduced.
- Parallelism is rate-limited to avoid hammering defuddle or rate-limit-sensitive sources.
