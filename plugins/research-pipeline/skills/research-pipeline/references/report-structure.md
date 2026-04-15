# Report Structure Reference

## HTML Report Sections

Every research report must include these sections in order:

### 1. Navigation
- Sticky nav bar at top
- Links to all sections below
- Brand: "Research Pipeline" with accent color on "Pipeline"

### 2. Hero / Title
- Topic name as h1
- Date, mode (Quick/Detailed), source count as metadata
- Categories as tags/badges

### 3. Executive Summary
- 2-3 paragraphs — the TL;DR
- Written for someone who will only read this section
- Include the single most important finding and the overall verdict preview

### 4. Key Facts
- Bulleted list with source URLs
- Preserve original data, quotes, and numbers
- Group by subtopic if >10 facts
- Each fact attributed: "Fact text (Source: [name](url))"

### 5. Market Research
- Competitive landscape, trends, market size (if applicable)
- Skip this section if the topic isn't market-relevant
- Include specific numbers and comparisons

### 6. Technical Analysis
- Tools, implementations, architectures (if applicable)
- Skip this section if the topic isn't technical
- Include code examples, stack comparisons, or architecture diagrams if relevant

### 7. Skeptic Analysis
- Full skeptic agent output, formatted
- Section header in warm brick `#A14841` (harmonizes with the coral accent — do not use pure red)
- Subsections: Overly Optimistic Claims, Hidden Risks, What Would Make This Fail, Realistic Assessment

### 8. Optimist Analysis
- Full optimist agent output, formatted
- Section header in muted forest `#4A6B4F` (warm earth-tone green — do not use neon/lime)
- Subsections: Core Opportunity, Supporting Evidence, Success Conditions, Actionable Implications

### 9. Debate Transcript
- Round-by-round exchange (only in Detailed mode)
- Each round shows: who spoke, key claims, evidence cited
- Highlight concessions and agreements

### 10. Verdict
- Parent synthesis with confidence rating (percentage)
- Points of agreement (high confidence findings)
- Points of disagreement (with both perspectives)
- Actionable next steps

### 11. Sources
- All URLs with brief descriptions
- Organized by type: primary sources, analysis, data, video
- Include access date

## Design Reference — Anthropic theme (default)

This is the default theme for every report. Inspired by Anthropic's editorial visual language — warm, intellectual, restrained. To override, the user must explicitly say "use a different theme" before generation.

### Palette

| Token | Hex | Use |
|---|---|---|
| `--bg` | `#F5F1EA` | Page background (warm cream) |
| `--bg-raised` | `#FAF7F1` | Card / inset surfaces |
| `--ink` | `#141413` | Body text (warm coal) |
| `--ink-muted` | `#5F5B54` | Secondary text |
| `--ink-faint` | `#8B8680` | Metadata, dividers' labels |
| `--rule` | `#E4DDD0` | Hairline rules |
| `--rule-soft` | `#EFEAE0` | Inner card borders |
| `--coral` | `#D97757` | Primary accent (Claude coral) — links, section numbers, italic flourishes |
| `--coral-deep` | `#BF5F3F` | Hover states |
| `--brick` | `#A14841` | Skeptic accent (warm red, harmonized with coral) |
| `--forest` | `#4A6B4F` | Optimist accent (warm earth-tone green) |

Optional supporting tones for facts/caution callouts: muted blue `#4A5E6B` for facts, warm amber `#A07242` for caution. Use sparingly — palette restraint is core to the theme.

### Typography

- **Display / headings:** **Fraunces** (variable, optical-size axis). Use `opsz: 144, weight: 400` for h1; `opsz: 72, weight: 500` for h2; `opsz: 36, weight: 500` for h3/h4. Italic variant for editorial flourishes (a single italic period after the title, italic accent words, etc.)
- **Body:** **Source Serif 4** (variable, optical-size axis). 16-17px, line-height ~1.65. Use Source Sans 3 only for dense tabular data where serif legibility suffers.
- **Mono:** **JetBrains Mono** (or IBM Plex Mono) for code, metadata strips, section numbers (`§ 01`, `§ 02`...), eyebrow labels, paths.

### Composition

- Narrow centered reading column (~680px max)
- Section labels: `§ 01` / `§ 02` style in JetBrains Mono, coral
- Hairline dividers between sections, no boxy cards by default
- Generous vertical whitespace (64px+ between sections)
- Page-load stagger animation (subtle: 14px rise + opacity fade, 80ms increments)
- Decorative typographic mark in footer (e.g. `※`, `§`, `◇`) in italic Fraunces, coral
- Selection color: coral on cream

### What this theme is NOT

- Not a generic dashboard with sidebar nav and stat cards
- Not a purple-gradient SaaS marketing page
- Not a Bootstrap-style card grid
- Not a dark-mode editor look — this is light-mode warm cream only

### Invocation

Always generate via `/frontend-design:frontend-design`. Pass this file (or the relevant section) to the skill as the design reference. Do NOT hand-code HTML and do NOT substitute the theme without explicit user request.
