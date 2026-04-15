# Research Pipeline Plugin

Research any topic with web search, content extraction, adversarial debate, and Obsidian vault integration. Say "research X" or run `/research <topic>` and get a polished HTML report with a Skeptic vs Optimist debate, automatically stored in your Obsidian vault.

---

## Quick Start

### 1. Set Your Vault Root

```bash
export RESEARCH_VAULT_ROOT="$HOME/my-vault"
```

The plugin reads `$RESEARCH_VAULT_ROOT` at runtime. If unset, it defaults to `~/Research/`. Add the export to your shell profile to persist it.

### 2. Basic Usage

```
/research database reactivation tools 2026
```

That's it. The pipeline searches the web, extracts content, runs a Skeptic/Optimist debate, generates an HTML report, and saves everything to your vault. Default mode is `--detailed` (3-5 debate rounds).

### 3. Modes

| Mode | Command | Debate Rounds | Best For |
|------|---------|--------------|----------|
| Detailed | `/research <topic>` | 3-5 rounds | Deep dives, business decisions |
| Quick | `/research <topic> --quick` | 1 round | Quick scans, time-sensitive lookups |

### 4. Input Types

```bash
# Topic search — finds sources automatically
/research AI agent frameworks 2026

# Direct URLs — analyzes specific pages
/research https://example.com/article https://youtube.com/watch?v=abc123

# Mixed — searches AND analyzes your URLs
/research speed to lead https://example.com/case-study --detailed
```

YouTube URLs are handled automatically — transcripts are pulled via `youtube-transcript-api`.

### 5. Natural Language (No Slash Command Needed)

The skill auto-triggers on phrases like:
- "Research the latest on..."
- "Look into..."
- "Investigate..."
- "What's the latest on..."
- "Find out about..."
- "Analyze this: [url]"

---

## What It Does

### The Pipeline (6 Phases)

```
Phase 0: Prerequisite Check
  └─ Verifies tools (youtube-transcript-api, defuddle) and vault structure

Phase 1: Source Collection
  └─ WebSearch for 3-5 sources, or fetches your URLs directly
  └─ Fallback chain: defuddle → WebFetch → mark unprocessable

Phase 2: Content Extraction
  └─ Pulls key quotes, data points, statistics, frameworks
  └─ Preserves original data — no paraphrasing

Phase 3: Adversarial Debate
  └─ Skeptic agent (Sonnet, brick) — finds risks, bias, failure modes
  └─ Optimist agent (Sonnet, forest) — finds genuine opportunity with evidence
  └─ Quick: 1 parallel round | Detailed: 3-5 rounds with cross-examination
  └─ Parent orchestrates, compresses between rounds, writes final verdict

Phase 4: Report Generation
  └─ HTML report via /frontend-design:frontend-design (Anthropic theme: warm cream + Claude coral, Fraunces + Source Serif 4 + JetBrains Mono)
  └─ Markdown companion with YAML frontmatter
  └─ 11 sections: Executive Summary → Key Facts → Market Research →
     Technical Analysis → Skeptic → Optimist → Debate Transcript →
     Verdict → Sources

Phase 5: Vault Integration
  └─ Classifies into categories (max 3)
  └─ Smart-merges facts into overview.md (append-only, dedup by URL)
  └─ Saves HTML to Research/reports/, updates dashboard
```

### The Debate

The adversarial debate is the core differentiator. Two Sonnet agents argue:

**Skeptic** asks:
- Who benefits from publishing this? Vendor-funded research?
- Are the stats cherry-picked or real?
- What are the hidden costs and failure modes?
- What would make this fail completely?

**Optimist** asks:
- What's the genuine, data-backed opportunity?
- What conditions make this succeed?
- Where does this fit in the market trend?
- What's the actionable next step this week?

In detailed mode, they cross-examine each other for 3-5 rounds, explicitly conceding or challenging points. The parent synthesizes an Agreement Zone, Disagreement Zone, and Verdict with a confidence percentage.

---

## How It Works (Architecture)

### Components

```
plugins/research-pipeline/
├── .claude-plugin/plugin.json          # Plugin manifest
├── commands/research.md                # /research slash command
├── commands/research-batch.md          # /research-batch slash command
├── commands/research-lint.md           # /research-lint slash command
├── agents/
│   ├── research-skeptic.md             # Skeptic debate agent (Sonnet)
│   ├── research-optimist.md            # Optimist debate agent (Sonnet)
│   └── research-router.md              # Routing decision agent (Sonnet)
└── skills/research-pipeline/
    ├── SKILL.md                        # Core methodology (auto-triggers)
    └── references/
        ├── report-structure.md         # 11-section HTML report spec
        ├── category-map.md             # 7 categories + classification rules
        ├── debate-protocol.md          # Round structure + convergence rules
        ├── router-logic.md             # Router priority + threshold rules
        ├── compounding.md              # Entity page + overview update flow
        ├── wiki-schemas.md             # Frontmatter schemas for all file types
        ├── batch-mode.md               # Batch-mode differences
        └── lint-checks.md              # 9 wiki health checks
```

### Vault Structure

```
$RESEARCH_VAULT_ROOT/Research/
├── AI-and-Agents/          overview.md + sources.md
├── Claude-Code/            overview.md + sources.md
├── Tools-and-Platforms/    overview.md + sources.md
├── Automations-and-Workflows/  overview.md + sources.md
├── Business-and-GTM/       overview.md + sources.md
├── Content-and-Media/      overview.md + sources.md
├── PKM-and-Obsidian/       overview.md + sources.md
├── reports/                HTML reports land here
├── my-ideas/               Your own ideas (separate from research)
├── _raw/                   Immutable cleaned-markdown copies of every ingested source
├── _entities/              Flat cross-cutting entity pages
├── _queries/               Filed-back answers to synthesis questions
├── _log.md                 Chronological timeline of ingests and lint passes
├── _lint-reports/          Dated wiki-health reports
├── _index.md               Dashboard with links to all categories
└── _inbox.md               Drop URLs for manual processing later
```

### Categories

| Category | What Goes Here |
|----------|---------------|
| AI & Agents | LLMs, agent frameworks, RAG, embeddings |
| Claude Code | Plugins, skills, MCP, Claude Code workflows |
| Tools & Platforms | SaaS, APIs, no-code builders, new releases |
| Automations & Workflows | n8n, Make, pipelines, MCP servers |
| Business & GTM | Sales, pricing, consulting, go-to-market |
| Content & Media | Video, ads, podcasts, content pipelines |
| PKM & Obsidian | Note-taking methodology, vault strategies |

Each research item gets up to 3 categories. Primary = where the file lives. Secondary/tertiary get cross-references.

---

## Prerequisites

- **youtube-transcript-api** — `pip3 install youtube-transcript-api` (for YouTube URLs)
- **frontend-design skill** — installed via Superpowers plugin (for HTML reports)
- **Vault structure** — created automatically on first run
- **`$RESEARCH_VAULT_ROOT`** — set to your vault directory (see Quick Start)

---

## Examples

### Quick scan of a trending topic
```
/research MCP server ecosystem 2026 --quick
```
→ 1 round debate, HTML report, vault merge. Fast.

### Deep dive for a business decision
```
/research database reactivation tools for SMBs --detailed
```
→ 3-5 round debate, full adversarial analysis.

### Analyze a specific YouTube video
```
/research https://www.youtube.com/watch?v=dQw4w9WgXcQ --quick
```
→ Pulls transcript, extracts key points, runs debate on claims.

### Mixed sources with a topic
```
/research AI ad generation https://example.com/ai-ads-study --detailed
```
→ Searches web for more sources AND analyzes your URL.

---

## Commands

### `/research <url-or-topic> [--quick|--detailed]`

Single-source research pipeline. Fetches, cleans, adversarial-debates, synthesizes, and integrates into the vault with full compounding (entity pages, overview updates, log). Default mode.

### `/research-batch <urls|file|folder|rss> [--parallelism=N] [--html-reports]`

Batch ingestion. Same pipeline as `/research` minus the debate, plus parallel fetching, resume via hash-dedupe, end-of-batch confirmation for ambiguous routings, and automatic lint on affected topics.

### `/research-lint [--scope=<topic|all>] [--auto-fix]`

Wiki health check. Scans `Research/` for orphans, missing entity pages, broken wikilinks, empty overviews, unprocessed raw, contradictions, stale claims, duplicate entities, and data gaps. Writes a dated report to `Research/_lint-reports/`.

## Wiki Spine

The pipeline maintains a wiki spine under `Research/`:

- `_raw/` — immutable cleaned-markdown copies of every ingested source
- `_entities/` — flat cross-cutting entity pages (people, tools, concepts, orgs, places, products)
- `_queries/` — filed-back answers to synthesis questions
- `_log.md` — chronological timeline of ingests, queries, lint passes, batches
- `_lint-reports/` — dated wiki-health reports

All entity pages and the compounding step update on every ingest. See `skills/research-pipeline/references/` for the full wiki schemas, router logic, compounding flow, batch-mode differences, and lint checks.
