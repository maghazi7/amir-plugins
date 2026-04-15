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
- Section header with red accent/icon
- Subsections: Overly Optimistic Claims, Hidden Risks, What Would Make This Fail, Realistic Assessment

### 8. Optimist Analysis
- Full optimist agent output, formatted
- Section header with green accent/icon
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

## Design Reference
- Fonts: Fraunces (headings) + Source Sans 3 (body)
- Primary accent: terracotta (#c2704e)
- Background: warm paper (#faf8f4)
- Cards with subtle shadows, rounded corners (10px)
- Color coding: red for skeptic, green for optimist, blue for facts, yellow for caution
- Responsive — readable on mobile
- Use the `frontend-design` skill to generate — do NOT hand-code HTML
