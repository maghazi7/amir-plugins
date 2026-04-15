# Compounding Reference

The compounding step runs after synthesis writes and before the final user report. It takes one ingest and fans it out across entity pages, topic/project overviews, cross-references, the raw file, and the log. Typical ingest touches 3–8 entity pages + 1–2 overviews.

Compounding is token-bounded by design: it never re-reads all sources in a topic. It reads only the new raw + current overview + entity pages that match entities in the new source.

## Inputs per ingest

Assemble before starting compounding:

- `raw_path`: `Research/_raw/<slug>.md` from Phase 2.5
- `synthesis_path`: whatever Phase 4/5 wrote (could be `Research/<topic>/...`, `Projects/<name>/research/...`, `Intelligence/competitors/<name>.md`, or `Intelligence/market/<topic>.md`)
- `extracted_entities`: list from Phase 2 content extraction, each with `{slug, entity_type, first_mention_context}`
- `extracted_facts`: list of `{fact, citation_hint}` pairs from Phase 2
- `topic_overview_path`: if synthesis is a topic/project/market file that has an `overview.md` sibling, that path; otherwise null
- `secondary_topics`: from router output — topics that get entity-backlink-only updates

## Step-by-step flow

### 1. Entity fan-out

For each entity `E` in `extracted_entities`:

1. **Resolve entity**: scan `Research/_entities/` for:
   - File named `<E.slug>.md` (exact match)
   - Any file whose frontmatter `aliases` contains `E.slug` or a known alternate
   - LLM fuzzy match on near-duplicate names (e.g., "NotebookLM" vs "Notebook LM") — only if the first two fail

2. **If entity page exists**:
   a. Read the existing page
   b. Append new facts to `## Key Facts` section with `[[_raw/<slug>]]` citations
   c. Append raw file to `## Sources` section as `- [[_raw/<slug>]] — <one-line how it appears>`
   d. Append synthesis file to `## Appears In` as `- [[<synthesis path>]]`
   e. Re-synthesize `## Summary` using bounded prompt: `existing summary + new facts → updated summary` (no full source re-read)
   f. **Contradiction check**: LLM compares new facts against existing Key Facts on this entity. If any conflict is detected (different dates, numbers, or claims), append a `## Contradictions` entry with both citations. Never auto-resolve.
   g. Increment frontmatter `source_count` by 1
   h. Update frontmatter `last_updated` to today
   i. Add any new topic slugs to frontmatter `topics` (union, no duplicates)

3. **If entity page does NOT exist**: create new `Research/_entities/<E.slug>.md` from `Resources/templates/entity-page-template.md`:
   - Frontmatter: `type: entity`, `entity_type: <E.type>`, `first_seen: <today>`, `last_updated: <today>`, `source_count: 1`, `topics: [<current topic>]` (union of primary + any secondary topics where this entity is the bridging link), `aliases: []`
   - Body:
     - `## Summary`: seed 1–2 sentence synthesis from how the entity appears in this source
     - `## Key Facts`: facts from this source with `[[_raw/<slug>]]` citations
     - `## Relationships`: infer from source if possible; leave empty otherwise
     - `## Sources`: `- [[_raw/<slug>]] — <one-line>`
     - `## Appears In`: `- [[<synthesis path>]]`
     - `## Contradictions`: empty (included for schema consistency with the template)

### 2. Overview update (primary destination)

**Intelligence file handling:** If the synthesis landed in `Intelligence/competitors/` or `Intelligence/market/`, apply entity-page-style compounding to the destination file instead of overview-style: append to `## Key Facts` with citations, re-synth `## Summary` (bounded), update `## Sources` and `## Appears In`, run contradiction check — same mechanics as entity pages in step 1, applied to the intelligence file per wiki-schemas.md section 7. Then skip to step 3.

If the synthesis landed in a topic or project folder that has an `overview.md` sibling:

1. **Read the overview**
2. **Normalize to canonical schema**: if the overview uses the legacy `## Key Facts` + `## Narrative Summary` format (common on existing topic overviews), normalize on first compounding touch: rename `## Narrative Summary` → `## Summary` (preserving its content), flatten nested `### <Entity>` subheadings under `## Key Facts` into flat bullet points with entity wikilinks, and add missing canonical sections (`## Key Entities`, `## Open Questions`, `## Contradictions`, `## Recent Sources`). All existing content is preserved — only heading structure changes.
3. **Append new Key Facts** with `[[_raw/<slug>]]` citations. Merge duplicates.
4. **Re-synthesize `## Summary`**: bounded prompt `existing summary + new facts from this source → updated summary`. (Legacy `## Narrative Summary` was already renamed to `## Summary` in step 2.)
5. **Update Key Entities**: add any newly introduced entities with their source counts.
6. **Append to Recent Sources**: `- YYYY-MM-DD — [[_raw/<slug>]] — <one-line>`. Trim the list to the most recent 10.
7. **Contradiction check** across overview Key Facts: append conflicts to `## Contradictions`.
8. **Bump frontmatter**: `last_updated: <today>`, `source_count += 1`. If `last_updated` or `source_count` fields don't exist yet (legacy overviews), initialize them (`last_updated: <today>`, `source_count: 1`).

### 3. Secondary topics (cross-reference only)

For each slug in `secondary_topics`:

1. Do NOT write a duplicate synthesis file.
2. The entity fan-out step in #1 already handles backlinks: any entity from this source that belongs to a secondary topic gets the synthesis file added to its `## Appears In`, and readers navigating from that topic's entity pages will find the source.
3. No direct modification of the secondary topic's `overview.md` in this phase — entity pages provide the backlink.

### 4. Sources.md append (existing behavior, preserved)

Append to the primary topic's `sources.md` (if the topic has one):

```
- [YYYY-MM-DD] [Title](URL) — Brief description. Report: [[reports/<slug>.html]]
```

Match the existing format exactly — do not reformat existing entries.

### 5. Update the raw file's frontmatter (exactly once)

Now that compounding is done, populate the raw file's `entities` and `synthesis_targets` fields. This is the only time these fields get written. After this step, the raw file is fully frozen.

```yaml
entities: [[entity-slug-a]], [[entity-slug-b]], [[entity-slug-c]]
synthesis_targets:
  - <synthesis file path>
  - Research/_entities/entity-slug-a.md
  - Research/_entities/entity-slug-b.md
  - Research/_entities/entity-slug-c.md
  - <topic overview path>
```

### 6. Update `Research/_index.md`

- **Recent Reports** section: add the new report as the top entry, matching the existing format.
- **Recent Entities** section: add any newly created entity pages as top entries (slug + one-line summary). Trim the list to the most recent 10.
- **Categories** section: only modified if a new category was created by the router in Phase 2.6.

### 7. Log to `Research/_log.md`

Append an ingest entry (see `references/wiki-schemas.md` section 4):

```
## [YYYY-MM-DD HH:MM] ingest | <source title>
Source: <url>
Synthesis: [[<synthesis path>]]
Raw: [[_raw/<slug>]]
Entities touched: [[entity-a]], [[entity-b]], [[entity-c]]
Pages updated: <count>
```

`Pages updated` count includes: 1 raw + 1 synthesis + N entity pages + (1 overview if updated) + (1 sources.md if updated) + 1 _index.md + 1 _log.md.

### 8. Report to user

Final user-visible line at the end of the `/research` response:

```
Ingested. Synthesis → <path>. Touched N entity pages (M new). K contradictions flagged. HTML: <path>.
```

## Token-bounding strategy

**Never re-read all sources in a topic.** Compounding reads only:
- The new raw file (just written)
- The current `overview.md` (if applicable)
- Entity pages matching entities extracted from the new source (usually 3–8)
- Nothing else

**Bounded Summary re-synthesis**: every LLM prompt for Summary updates has the shape `[existing Summary text] + [new facts from this source] → [updated Summary text]`. No full source re-reads.

**Cost envelope**: typical ingest = 10–30k input tokens, 5–15k output tokens. Scales with entities-per-source, not total-sources-ever.

## Contradiction detection

After writing new Key Facts to any entity or overview:

1. LLM compares new facts against existing Key Facts on the same page
2. Flag any conflict (different dates, different numbers, different claims) as a contradiction
3. Append entry to the page's `## Contradictions` section:
   ```
   - Source A says X (<date>) — [[_raw/source-a]]
     Source B says Y (<date>) — [[_raw/source-b]]
   ```
4. Never auto-resolve — user picks which source wins
5. Also log contradictions to the ingest entry in `_log.md`

## Failure modes

- **Entity resolution picks the wrong page**: surfaced later as a Duplicate Entities finding by `/research-lint`.
- **Compounding interrupted mid-way**: raw file was saved at Phase 2.5, so the source is preserved. The raw's empty `synthesis_targets` field surfaces as an Unprocessed Raw finding by `/research-lint`. Re-running compounding on an existing raw is supported — reads the raw body as input without re-fetching.
- **Overview re-synthesis produces bad output**: since the prompt is bounded to existing summary + new facts, the damage is local. User can manually edit the overview; nothing else breaks.

## What compounding does NOT do

- Re-read all sources ever ingested (token explosion)
- Auto-resolve contradictions
- Delete existing content (heading normalization in step 2 is the sole rename exception)
- Touch synthesis files from previous ingests (immutability)
- Propagate updates across multiple hops (entity A → entity B → entity C): the fan-out is exactly one level deep
