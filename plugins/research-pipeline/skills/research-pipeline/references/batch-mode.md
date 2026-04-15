# Batch Mode Reference

Full flow for `/research-batch`. Same skeleton as the single-URL pipeline in SKILL.md, with the differences documented here.

## Input normalization

Turn any input type into a list of URLs:

| Input | Handling |
|---|---|
| Space/newline-separated URLs | Parse directly |
| `.txt` / `.md` / `.list` file | Read lines, strip blanks and comments (`#` prefix) |
| Directory | For each file in the directory, treat as a local source (read content directly, skip fetch step) |
| RSS/Atom URL | Fetch feed, extract `<item>/<link>` or `<entry>/<link>` URLs |

Output: `urls: [<url-or-local-path>]`.

## Step-by-step

### 1. Dedupe pre-pass

For each URL:
- Scan `Research/_raw/` for matching `source_url` in frontmatter
- If found, mark as "already ingested" and move to the resume list (surfaced in the final report but not re-processed)

Remaining URLs → the processing queue.

### 2. Parallel fetch + clean

Dispatch browser-worker subagents with parallelism = N (default 4). Use session names `batch-1`, `batch-2`, ..., `batch-N`.

Rate limits:
- Max 5 concurrent browser-workers (hard ceiling, regardless of `--parallelism` value)
- One source per worker — do NOT batch multiple URLs into one worker
- If a source's URL indicates JS-heavy SPA (detected by trying defuddle first and checking for empty content), route to browser-worker directly

Each worker returns: `{title, cleaned_content, extracted_entities, extracted_themes, hash, source_type}` — the same shape the single-URL pipeline uses after Phase 2.

### 3. Raw save (per source, sequential)

Writes to `_raw/` are sequential — not parallel — to prevent slug collisions on same-day same-title sources. Each raw save uses the format from `references/wiki-schemas.md` section 1.

### 4. Router decision (per source)

For each source: invoke router logic per `references/router-logic.md`. In batch mode:
- Confidence ≥ 0.80: auto-route, proceed
- 0.60–0.79: queue as weak-match for end-of-batch confirmation (proceed with draft status)
- New category: queue in the proposal bucket �� defer synthesis and compounding for this source until after step 8 confirmation (the destination folder doesn't exist yet)
- < 0.60 everywhere: route to `Research/_inbox.md` with batch note, skip synthesis

**New category auto-create threshold**: if ≥ 3 sources in the batch propose the same new category slug, auto-create it mid-batch (don't wait for confirmation). Sources with auto-created categories proceed immediately to synthesis.

**Post-confirmation loop-back**: after step 8 resolves new-category proposals, run steps 5-6 (synthesis + compounding) for any previously deferred sources whose categories are now confirmed. Sources whose categories were rejected go to `_inbox.md`.

### 5. Straight synthesis (per source)

Skip the adversarial debate. The synthesis is a direct LLM summary of the cleaned content, 400–800 words, with inline citations. Use the same markdown companion format from Phase 4 but omit the Skeptic/Optimist/Debate/Verdict sections. Frontmatter still gets set, including `mode: batch`.

### 6. Compounding (per source, sequential)

Compounding writes are sequential — not parallel — to prevent concurrent writes to the same entity page. Follow `references/compounding.md` exactly for each source. This is the bottleneck of batch mode; it's acceptable because entity pages are small and compounding is token-bounded.

### 7. HTML report (optional, per source)

Only if `--html-reports` flag is set. Skipped by default in batch mode for throughput.

### 8. End-of-batch confirmation round-trip

Surface to the user in one message:
- **Weak-match routings**: list each with source + destination + confidence; ask "accept all / reject all / review individually"
- **New category proposals**: list each with slug + description + proposing sources; ask "auto-create all / reject all / review individually"

Apply user decisions, then proceed to step 9.

### 9. Auto-lint on affected topics

Run `/research-lint` scoped to:
- Every topic/project/intelligence folder touched by this batch
- Every entity page touched by this batch

Write the lint report to `Research/_lint-reports/YYYY-MM-DD-batch-<timestamp>-lint.md`.

### 10. Log entry

Append batch entry to `Research/_log.md`:

```
## [YYYY-MM-DD HH:MM] batch | <count> sources
Sources: <count>
New entities: <count>
New categories: <count> (auto: <n>, confirmed: <m>)
Contradictions: <count>
Failures: <count>
Auto-lint: [[_lint-reports/<slug>]]
```

### 11. Final report to user

Present the batch summary message (see `commands/research-batch.md` Final batch report section).

## Resume behavior

Re-running `/research-batch` with the same input set uses the hash-dedupe in step 1 to skip already-processed sources. The resume list in the final report tells the user what was skipped and why.

## Failure handling

| Failure | Behavior |
|---|---|
| Single source fetch fails | Log to failure list, continue batch |
| Router can't decide on a source | Route to `_inbox.md`, note in batch report |
| Compounding fails for a source | Raw is preserved, synthesis file may or may not exist; surfaces as Unprocessed Raw in auto-lint |
| Browser-worker subagent crashes | Retry once; on second failure, add to failure list |

Failures are reported at end of batch — batch mode never aborts on a single source's failure.
