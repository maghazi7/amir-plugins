# Lint Checks Reference

Full specification for every check run by `/research-lint`. Each check has: what it detects, how to detect, whether it auto-fixes, and what appears in the report.

## Scope

Without `--scope`, lint runs across all of `$RESEARCH_VAULT_ROOT/Research/` (default `~/Research/`):
- `_raw/` — every raw file
- `_entities/` — every entity page
- `_queries/` — every query page
- Every topic folder (`AI-and-Agents/`, `Claude-Code/`, etc.) — their `overview.md`, `sources.md`, and dated synthesis files
- `_index.md`, `_log.md`

Lint does NOT scan:
- `Projects/` (project research is owned by projects, not the wiki lint)
- `Intelligence/` (intelligence files have different schemas)
- `_lint-reports/` (lint doesn't lint itself)

Scoped lint (`--scope=<topic>`) runs the topic-applicable checks on the given topic folder plus any entity pages touched by its synthesis files.

## Check 1: Orphans

**Detects:** entity pages or synthesis files with zero inbound `[[wikilinks]]` from any other wiki file.

**Detection:**
1. Build an index of all `[[wikilink]]` targets across `Research/` (including `_index.md`, overviews, sources.md, synthesis files, entity pages, and `_log.md`)
2. For each entity page and synthesis file, check if its slug appears as a wikilink target
3. Files with zero inbound links are orphans

**Auto-fix:** No. Orphans may be WIP — user should decide.

**Report format:**
```
## Orphans (N)
- [[Research/_entities/<slug>]] — no inbound references
- [[Research/<topic>/<date>-<slug>]] — no inbound references
```

## Check 2: Missing entity pages

**Detects:** names/concepts mentioned in synthesis files or entity pages' Relationships sections that don't have a corresponding `_entities/<slug>.md`.

**Detection:**
1. Scan every synthesis file and every entity page `## Relationships` section for `[[<slug>]]` wikilinks whose target does not exist under `Research/_entities/`
2. Exclude `_raw/`, `_queries/`, topic overview, and `_log.md` targets — those are legitimate non-entity pages
3. Also scan synthesis files for text that looks like a proper-noun mention (capitalized, 2+ words, not in a `[[wikilink]]` already) — flag LLM-detected candidates

**Auto-fix (with `--auto-fix`):** create a seed entity page using `Resources/templates/entity-page-template.md` with:
- `entity_type`: inferred by LLM from context
- `Summary`: first-mention context snippet
- `Key Facts`: the sentence containing the first mention, with citation
- `first_seen`: today
- `source_count`: 1

**Report format:**
```
## Missing Entity Pages (N)
- `<slug>` — first mentioned in [[<source>]] (auto-fixed: created seed page)
- `<slug>` — mentioned in [[<source>]] (awaiting review)
```

## Check 3: Broken wikilinks

**Detects:** `[[target]]` references whose target doesn't resolve to any file in the vault.

**Detection:**
1. For every wikilink in every wiki file, check if the target resolves
2. Check both exact-slug matches and alias matches (via entity page frontmatter)

**Auto-fix (with `--auto-fix`):**
- If the target exists at a different slug (typo / rename), rewrite the wikilink to the correct slug
- If the target matches an alias in an entity page, rewrite to the canonical slug
- Otherwise, leave the broken link and report it

**Report format:**
```
## Broken Wikilinks (N)
- [[missing-slug]] in [[<source-file>]] (auto-fixed → [[correct-slug]])
- [[another-missing]] in [[<source-file>]] (no resolution found)
```

## Check 4: Empty overviews

**Detects:** topic `overview.md` files whose body contains "No facts yet" or is otherwise empty of structured content, but whose topic folder has sources (either in `sources.md` or dated synthesis files).

**Detection:**
1. For each topic folder under `Research/`:
   - Read `overview.md` body
   - If body contains "No facts yet" OR has zero `## Key Facts` entries AND the topic has ≥ 1 source in `sources.md` or any dated synthesis files, flag it

**Auto-fix (with `--auto-fix`):** re-run compounding using every dated synthesis file in the topic as input. This is a batch-compound operation limited to that topic.

**Report format:**
```
## Empty Overviews (N)
- `Research/PKM-and-Obsidian/overview.md` — 0 facts, 0 sources (genuinely empty, no fix needed)
- `Research/Claude-Code/overview.md` — 0 facts, 3 sources (auto-fixed: re-ran compounding)
```

## Check 5: Unprocessed raw

**Detects:** `_raw/` files with empty `synthesis_targets` frontmatter — indicates an interrupted ingest.

**Detection:**
1. For each file in `Research/_raw/`, parse frontmatter
2. If `synthesis_targets: []` or field is missing, flag it

**Auto-fix (with `--auto-fix`):** offer to re-run compounding on the raw file. Requires reading the raw body, extracting entities, and invoking the router + compounding flow. In `--auto-fix` mode, proceed without asking.

**Report format:**
```
## Unprocessed Raw (N)
- [[_raw/<slug>]] — empty synthesis_targets (auto-fixed: re-compounded → <destination>)
```

## Check 6: Contradictions

**Detects:** conflicting facts across entity pages and overviews. LLM-judged.

**Detection:**
1. Scan every entity page and overview for `## Contradictions` sections already flagged by the compounding step
2. Additionally: LLM cross-reads Key Facts within each entity page for contradictions that haven't been flagged yet

**Auto-fix:** No. Contradictions require user judgment on source reliability.

**Report format:**
```
## Contradictions (N)
- [[_entities/<slug>]]: Source A says X, Source B says Y
  - [[_raw/source-a]]
  - [[_raw/source-b]]
```

## Check 7: Stale claims

**Detects:** entity pages with `last_updated` older than 30 days AND a newer raw source that mentions the entity but wasn't compounded into the page (usually because of an interrupted ingest or a historical gap).

**Detection:**
1. For each entity page, read `last_updated`
2. If `last_updated` > 30 days ago:
   - Scan `_raw/` for files with `date` > entity's `last_updated` whose body mentions the entity's slug or aliases
   - If matches found, flag as stale

**Auto-fix:** No. Suggest the user re-run `/research` or `/research-batch` scoped to the affected raw files.

**Report format:**
```
## Stale Claims (N)
- [[_entities/<slug>]] — last_updated 2026-02-15, newer raw: [[_raw/2026-03-10-source]]
```

## Check 8: Duplicate entities

**Detects:** entity pages that look like aliases of the same thing (different slugs, near-identical summaries or aliases).

**Detection:** LLM pairwise comparison of entity page summaries + aliases. Heuristics:
- Normalized names match (e.g., "NotebookLM" vs "Notebook LM" vs "notebook-lm")
- Aliases overlap
- Summaries cover the same concept

**Auto-fix:** No. Merges are destructive and require user confirmation on which slug wins.

**Report format:**
```
## Duplicate Entities (N)
- [[_entities/notebook-lm]] and [[_entities/notebooklm]] appear to be duplicates
  - Suggested merge: keep `notebook-lm`, archive `notebooklm`
```

## Check 9: Data gaps / suggestions

**Detects:** areas of the wiki that need more research:
- Entity pages with `source_count: 1` (could use corroboration)
- Topic overviews with `source_count < 3` (thin evidence)
- Open Questions sections in overviews that haven't been resolved
- Entity pages whose Relationships section is empty despite multiple sources

**Detection:** frontmatter scan + section-content scan.

**Auto-fix:** No. Report only, as research suggestions.

**Report format:**
```
## Data Gaps / Suggestions (N)
- [[_entities/<slug>]] has only 1 source — consider additional research
- [[Research/Voice-AI-and-Audio/overview.md]] has 2 sources — thin evidence
- [[Research/AI-and-Agents/overview.md]] has open question: "How does framework X handle streaming?" (no sources address this)
```

## Report file

Write to `Research/_lint-reports/YYYY-MM-DD-lint.md` (or `YYYY-MM-DD-batch-<timestamp>-lint.md` if invoked from a batch run).

Report structure: YAML frontmatter (see `references/wiki-schemas.md` section 6) + all 9 check sections in order, with counts in headings.

## Log entry

Append a lint entry to `Research/_log.md`:

```
## [YYYY-MM-DD HH:MM] lint | <scope>
Report: [[_lint-reports/<slug>]]
Found: <orphans>, <contradictions>, <missing entities>, <stale>
Auto-fixed: <count if --auto-fix was set, else 0>
```
