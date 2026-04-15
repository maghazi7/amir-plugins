# Router Logic Reference

The router runs between content extraction and synthesis in the research pipeline. It's implemented as a dedicated subagent (`agents/research-router.md`) that receives structured inputs and returns a routing decision. The pipeline skill delegates to this subagent at Phase 2.6. This reference defines the logic the subagent follows.

Priority + threshold interaction is the subtle part: a higher-priority match only wins if it meets the high-confidence threshold (≥ 0.80). A weak high-priority match falls through to the next priority level. See Worked Examples below.

## Decision priority

Top wins, but only if confidence ≥ 0.80 at that level:

```
1. PROJECT MATCH        → Projects/{name}/research/{YYYY-MM-DD}-{slug}.md
2. COMPETITOR MATCH     → Intelligence/competitors/{name}.md
3. MARKET/TREND MATCH   → Intelligence/market/{topic}.md
4. TOPIC RESEARCH       → Research/<topic>/{YYYY-MM-DD}-{slug}.md
```

## Priority + threshold interaction

- A priority level is "selected" only if its score is ≥ 0.80.
- If priority 1 scores 0.65 and priority 4 scores 0.85, the router falls through to topic (priority 4 wins because only it meets the threshold).
- If nothing reaches 0.80 but topic has a weak match (0.60–0.79), topic is used **with a weak-match flag** (synthesis written with `status: draft` in frontmatter).
- If the topic score is also < 0.60 but some other topic could fit with a new category (see "New category creation" below), propose the new category.
- If nothing reaches 0.60 on any level AND no new category is proposed, route to `Research/_inbox.md` with a "router couldn't decide" note.

**Rationale:** Projects drive actionable work (tasks and decisions), so they win when confident. Competitors are named entities (higher precision) while market is synthesized trends (lower precision). Topic is the catch-all. The threshold prevents a weak project signal from pulling a source away from a strong topic fit.

## Router inputs

Assemble this structured object before calling the router LLM step:

```yaml
source:
  title: <extracted title>
  url: <source url>
  cleaned_content: <first ~3k tokens of the cleaned markdown body>
  extracted_entities: [<entity slug>, <entity slug>, ...]  # from content extraction phase
  extracted_themes: [<theme>, <theme>, ...]  # from content extraction phase

vault_state:
  active_projects:
    - name: <project name>
      description: <one-line from Projects/{name}/README.md overview>
      keywords: [<keyword>, <keyword>, ...]
    # ... repeat for every active project found in Projects/ with status: active
  research_topics:
    - slug: <topic folder name>
      description: <from references/category-map.md>
      source_count: <from overview.md frontmatter, or 0 if missing>
    # ... one entry per folder under Research/ matching references/category-map.md
  competitors: [<slug>, ...]  # from Intelligence/competitors/ filenames
  market_topics: [<slug>, ...]  # from Intelligence/market/ filenames
```

Active projects are discovered via: `ls Projects/ | for each, grep "^status: active" */README.md`. Competitors and market topics are discovered via directory listings of `Intelligence/competitors/` and `Intelligence/market/`.

## Router LLM prompt

Feed the inputs above plus this instruction to the router step:

```
You are the research-pipeline router. Decide where a source's synthesis should live. You have four destination types in priority order: project (1), competitor (2), market (3), topic (4).

Score each applicable destination from 0.0 to 1.0. A score means: "how confident am I this source is primarily about this destination target?"

Apply priority + threshold rules:
1. A priority level wins ONLY if its best score ≥ 0.80.
2. If priority N does not reach 0.80, try priority N+1.
3. If no level reaches 0.80 but TOPIC has a match in [0.60, 0.80), use topic with a weak-match flag.
4. If topic's best score is < 0.60, propose a NEW topic (see new-category spec).
5. If every level scores < 0.60 AND no viable new category, route to inbox with "router couldn't decide".

Return structured output:

primary_destination: <path>
synthesis_filename: <YYYY-MM-DD-kebab-slug.md>
confidence: <0.00 to 1.00>
reasoning: <1-3 sentences>
secondary_topics: [<topic slug>, ...]  # entity backlinks cross-ref; no duplicate synthesis
new_category: null | { slug, display_name, description, first_source }
needs_confirmation: <true if confidence < 0.80 OR new_category is set>
weak_match_flag: <true if routing to topic with 0.60-0.79 score>
```

## Output format

```yaml
primary_destination: Research/AI-and-Agents/
synthesis_filename: 2026-04-05-seedance-2-access-us.md
confidence: 0.85
reasoning: >
  Primarily about a new video AI model and US access paths. Fits AI-and-Agents best. Mentions
  fal.ai (cross-ref to Tools-and-Platforms). Not project-specific, not a competitor.
secondary_topics:
  - Research/Tools-and-Platforms/
new_category: null
needs_confirmation: false
weak_match_flag: false
```

## New category creation

When no existing topic scores ≥ 0.60, propose a new category. Set `confidence` to the best rejected topic score (the highest score that didn't reach the threshold) — this tells the caller how close the source was to fitting an existing category:

```yaml
new_category:
  slug: Voice-AI-and-Audio
  display_name: "Voice AI & Audio"
  description: "Voice synthesis, TTS, audio-to-audio, podcast AI"
  first_source: [[_raw/2026-04-05-sesame-voice]]
needs_confirmation: true
```

- **Single mode** (`/research`): show the proposal, wait for yes/no. On yes: create folder + `overview.md` + `sources.md` from templates + append to `references/category-map.md` + update `Research/_index.md` Categories list.
- **Batch mode** (`/research-batch`): queue proposals. Auto-create if ≥ 3 sources in a batch propose the same new slug. Confirm any remaining queued proposals in one round-trip at end of batch.

New category on-disk creation:

```bash
mkdir -p "$RESEARCH_VAULT_ROOT/Research/<slug>"
```

Write `<slug>/overview.md`:
```yaml
---
type: research-overview
category: <slug>
last_updated: YYYY-MM-DD
source_count: 0
status: active
tags: [research, <slug>]
---

## Summary

## Key Entities

## Key Facts

## Open Questions

## Contradictions

## Recent Sources
```

Write `<slug>/sources.md`:
```yaml
---
type: research-sources
category: <slug>
date: YYYY-MM-DD
status: active
tags: [research, sources, <slug>]
---

<!-- Sources are appended automatically by the research pipeline. -->
```

Append to `references/category-map.md` (see existing table format in that file).

## Confirmation rules

| Situation | Single mode | Batch mode |
|---|---|---|
| Confidence ≥ 0.80, existing destination | Auto-route | Auto-route |
| Confidence 0.60–0.79, topic weak match | Ask user once | Queue; batch-confirm at end |
| New category proposed | Ask user once | Queue; auto-create if 3+ sources in batch agree, else batch-confirm |
| Confidence < 0.60 everywhere | Route to `_inbox.md` with note | Same, add to batch failure list |

## Edge cases

| Case | Behavior |
|---|---|
| Multi-topic source (fits 2+ topics well) | Primary route wins the synthesis file. Secondary topics get cross-reference via entity page backlinks — no duplicate synthesis. |
| Project AND topic both strong | Project wins (priority 1). Topic overview still updated via entity backlinks. |
| Source is about a competitor to a Project | Routes to `Intelligence/competitors/{name}.md`. Entity backlink surfaces it when reading the Project's research folder. |
| Low-quality source (thin, promotional) | Save raw as normal. Mark synthesis `status: draft` in frontmatter and ask user whether to promote. |
| Source duplicates an existing raw hash | Abort with pointer to existing raw. Offer re-run compounding. |

## Worked examples

### Example 1: Strong project match

Source: `https://example.com/construction-ai-workflow-automation`

```
Scores: project(My Project)=0.92, topic(Automations)=0.78, topic(AI-and-Agents)=0.65
Priority 1 hits threshold → project wins.
primary_destination: Projects/my-project/research/
secondary_topics: [Automations-and-Workflows]  # backlink only
```

### Example 2: Priority fallthrough

Source: `https://arxiv.org/abs/2404.abcd-agent-transformer-attention`

```
Scores: project(any)=0.30, competitor(any)=0.10, market=0.15, topic(AI-and-Agents)=0.91
Priorities 1-3 all < 0.80 → fall through to topic.
primary_destination: Research/AI-and-Agents/
```

### Example 3: Weak topic

Source: `https://example.com/generic-startup-tips`

```
Scores: project=0.20, competitor=0.05, market=0.25, topic(Business-and-GTM)=0.68, topic(others)<0.5
No level ≥ 0.80; topic in [0.60, 0.80) → weak-match flag.
primary_destination: Research/Business-and-GTM/
weak_match_flag: true
synthesis frontmatter: status: draft
```

### Example 4: New category

Source: `https://example.com/voice-ai-podcast-generation-tool`

```
Scores: project=0.15, competitor=0.05, market=0.30, topic(Content-and-Media)=0.52, topic(AI-and-Agents)=0.48
All topic scores < 0.60 → propose new category.
primary_destination: Research/Voice-AI-and-Audio/ (folder created by caller after confirmation)
new_category: { slug: "Voice-AI-and-Audio", display_name: "Voice AI & Audio", ... }
confidence: 0.52 (best rejected topic score — see note below)
needs_confirmation: true
```

### Example 5: Router can't decide

Source: `https://example.com/random-essay-on-creativity`

```
Scores: everything < 0.60, no coherent new category either.
primary_destination: Research/_inbox.md (with note "router couldn't decide for <title>")
```
