---
name: research
description: Research a topic with web search, content extraction, adversarial debate, and vault integration
argument-hint: <topic or URLs> [--quick|--detailed]
---

# /research Command

You are executing the research pipeline. Parse the arguments and invoke the methodology.

## Argument Parsing

Arguments are in `$ARGUMENTS`. Parse them as follows:

1. **Mode detection:**
   - If `--quick` is present: Quick mode (1-round debate, abbreviated processing)
   - If `--detailed` is present: Detailed mode (3-5 round debate, full processing)
   - Default (no flag): Detailed mode

2. **Source detection:**
   - If arguments contain URLs (starting with `http`): treat as direct source analysis
   - If arguments are a topic string: search web first to find sources
   - If mixed (topic + URLs): search for topic AND analyze provided URLs

3. Strip mode flags from the topic/URL list before processing.

## Execution

Invoke the `research-pipeline` skill to execute the full methodology:

0. **Prerequisite Check** — Verify tools and vault infrastructure
1. **Source Collection** — WebSearch for sources or fetch provided URLs directly
2. **Content Extraction** — Extract key facts, quotes, data points, frameworks
2.5. **Raw Save** — Persist cleaned source to `Research/_raw/<slug>.md` (immutable). Dedupe on hash + URL.
2.6. **Route** — Router decides destination (project / competitor / market / topic). See `references/router-logic.md`.
3. **Adversarial Debate** — Launch `research-skeptic` and `research-optimist` agents (single mode only)
4. **Synthesis + Report** — Generate HTML report (with debate output) using `frontend-design` skill + markdown companion
5. **Vault Integration** — Write synthesis to routed destination, run compounding (entities, overview, cross-refs, raw frontmatter, log). See `references/compounding.md`.

## Important Notes

- **ALL primary web content extraction uses Playwright CLI** via browser-worker subagents — WebFetch is not used; defuddle is the fallback only when Playwright fails
- WebSearch is still used for URL discovery (finding sources), but content extraction from those URLs always goes through Playwright
- For YouTube URLs, use `python3 -m youtube_transcript_api <VIDEO_ID>` for transcript extraction
- `defuddle parse <url> --md` is the fallback only when Playwright fails for a specific URL
- HTML reports use the `frontend-design` skill — do NOT hand-code HTML
- All vault files use Obsidian-flavored markdown with `[[wikilinks]]`
- Max 3 categories per research item (see `references/category-map.md`)
- Router decision runs BEFORE debate — see `references/router-logic.md`. Compounding runs AFTER synthesis — see `references/compounding.md`. Schemas in `references/wiki-schemas.md`.
