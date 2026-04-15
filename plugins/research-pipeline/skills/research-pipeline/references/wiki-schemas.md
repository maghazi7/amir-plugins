# Wiki Schemas Reference

Single source of truth for every file schema in the research wiki. The pipeline skill and the lint command both reference this file — do not duplicate schemas elsewhere.

## 1. Raw source — `Research/_raw/YYYY-MM-DD-<slug>.md`

```yaml
---
type: raw-source
date: 2026-04-05
source_url: https://example.com/article
source_title: "Original Title"
source_type: article | paper | video | pdf | transcript | tweet
hash: sha256:<64-hex-chars>
entities: [[entity-slug-a]], [[entity-slug-b]]
synthesis_targets:
  - Research/AI-and-Agents/2026-04-05-article.md
  - Research/_entities/entity-slug-a.md
tags: [ai-and-agents]
---

# Original Title

<cleaned markdown from defuddle or Playwright fallback>
```

**Rules:**
- Slug format: `YYYY-MM-DD-kebab-title` (truncate title to ~60 chars)
- `hash` = SHA-256 of cleaned content body — used for dedupe on re-ingest
- Content body immutable after write
- `entities` and `synthesis_targets` written exactly once by the compounding step, then frozen

## 2. Entity page — `Research/_entities/<slug>.md`

```yaml
---
type: entity
entity_type: person | tool | concept | organization | place | product
aliases: ["Alt Name", "Acronym"]
first_seen: 2026-04-02
last_updated: 2026-04-05
source_count: 5
topics: [ai-and-agents, pkm-and-obsidian]
tags: [entity]
---
```

Body sections (in order): `## Summary`, `## Key Facts`, `## Relationships`, `## Sources`, `## Appears In`, `## Contradictions` (appears only when conflicts are flagged).

**Rules:**
- Flat layout — no subdirs under `_entities/`
- Slug = canonical name, kebab-case; aliases go in frontmatter
- Summary is LLM-maintained, re-synthesized on every ingest (bounded)
- Key Facts always have `[[_raw/...]]` citations
- Appears In auto-maintained by the compounding step

## 3. Topic overview — `Research/<topic>/overview.md`

```yaml
---
type: research-overview
category: <topic-slug>
last_updated: 2026-04-05
source_count: 12
status: active
tags: [research, <topic-slug>]
---
```

Body sections (in order): `## Summary`, `## Key Entities`, `## Key Facts`, `## Open Questions`, `## Contradictions`, `## Recent Sources`.

**Normalization rule:** Existing topic overviews (created by the older pipeline) use `## Key Facts` with nested subheadings + `## Narrative Summary`. On first compounding touch, the step **normalizes to canonical schema**: rename `## Narrative Summary` → `## Summary` (preserving content), flatten nested `### <Entity>` subheadings under `## Key Facts` into flat bullet points with entity wikilinks, and add any missing canonical sections (`## Key Entities`, `## Open Questions`, `## Contradictions`, `## Recent Sources`). Content under each heading is preserved — only the heading names and nesting change.

**Rules:**
- Summary re-synthesized on ingest (bounded: new source + current summary + affected entity pages, not full re-read)
- Key Facts append with citations; duplicates merged
- Contradictions flag-only, never auto-resolved
- Recent Sources = rolling last ~10 ingests

## 4. Log entry — `Research/_log.md` (append-only)

```markdown
## [2026-04-05 14:32] ingest | Accessing Seedance 2.0 from the US
Source: https://...
Synthesis: [[Research/AI-and-Agents/2026-04-05-seedance-2]]
Raw: [[_raw/2026-04-05-seedance-2-access-us]]
Entities touched: [[seedance-2]], [[bytedance]], [[fal-ai]]
Pages updated: 4

## [2026-04-05 14:10] query | How does Quality Score interact with AI Overviews?
Saved: [[_queries/2026-04-05-quality-score-ai-overviews]]

## [2026-04-04 22:15] lint | full pass
Report: [[_lint-reports/2026-04-04-lint]]
Found: 3 orphans, 1 contradiction, 5 missing entity pages, 2 stale
```

**Rules:**
- Every entry starts with `## [YYYY-MM-DD HH:MM]`
- Types: `ingest`, `query`, `lint`, `batch`
- Append-only
- `grep "^## \[" _log.md | tail -N` returns last N entries

## 5. Query file-back — `Research/_queries/YYYY-MM-DD-<slug>.md`

```yaml
---
type: query-answer
date: 2026-04-05
question: "How does Quality Score interact with AI Overviews?"
sources_used: [[_raw/2026-04-05-seo-for-ads]]
entities: [[quality-score]], [[ai-overviews]]
topics: [business-and-gtm, content-and-media]
---
```

Body sections: `## Question`, `## Answer` (with inline `[[_raw/...]]` citations), `## Sources Used`, `## Related Entities`.

## 6. Lint report — `Research/_lint-reports/YYYY-MM-DD-lint.md`

```yaml
---
type: lint-report
date: 2026-04-05
scope: research-wiki
tags: [lint, report]
---
```

Body sections (each with count in heading): `## Orphans (N)`, `## Missing Entity Pages (N)`, `## Broken Wikilinks (N)`, `## Empty Overviews (N)`, `## Unprocessed Raw (N)`, `## Contradictions (N)`, `## Stale Claims (N)`, `## Duplicate Entities (N)`, `## Data Gaps / Suggestions (N)`.

**Rules:**
- One report per lint run
- Dated filename, append as sibling to existing reports in `_lint-reports/`
- Auto-fixes applied BEFORE the report is written; report lists what was fixed vs what still needs attention

## 7. Intelligence file — `Intelligence/competitors/<name>.md` and `Intelligence/market/<topic>.md`

```yaml
---
type: intelligence
subtype: competitor | market-intel
aliases: ["Alt Name"]
first_seen: 2026-04-02
last_updated: 2026-04-05
source_count: 3
topics: [glp1, telehealth]
tags: [intelligence, competitor]
project: optional-linked-project
---
```

Body sections (same as entity pages): `## Summary`, `## Key Facts`, `## Relationships`, `## Sources`, `## Appears In`, `## Contradictions` (appears only when conflicts are flagged).

**Rules:**
- Uses entity-page-style schema — compounding treats these identically to entity pages (append Key Facts, re-synth Summary, update Sources/Appears In, detect contradictions)
- `subtype: competitor` for competitor files, `subtype: market-intel` for market files
- `project` field links to the most relevant project (optional)
- Existing intelligence files that use a different structure get normalized on first compounding touch (same as legacy overview normalization)
- Unlike `_entities/` pages, these live in their Intelligence subfolder — the router decides which goes where
