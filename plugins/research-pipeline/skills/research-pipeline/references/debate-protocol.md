# Debate Protocol Reference

## Overview

The adversarial debate pits a Skeptic agent against an Optimist agent, with the parent (Opus) orchestrating rounds and synthesizing the final verdict.

## Round Structure

### Round 1 (Parallel)
Both agents receive the same input simultaneously:
- Raw research findings (extracted content, key facts, data points)
- Topic context and mode (quick/detailed)

**Skeptic returns:** Critique with sections — Overly Optimistic Claims, Hidden Risks, What Would Make This Fail, Realistic Assessment
**Optimist returns:** Opportunity analysis with sections — Core Opportunity, Supporting Evidence, Success Conditions, Actionable Implications

### Round N (N >= 2, Sequential, Parent-Orchestrated)

Parent compresses both agents' Round N-1 output to structured bullet points:
```
## Summary of [Agent]'s Position (Round N-1)
- **Claim 1:** [claim] — Evidence: [cited source] — Confidence: [high/medium/low]
- **Claim 2:** [claim] — Evidence: [cited source] — Confidence: [high/medium/low]
- **Concessions:** [any points conceded from other side]
```

Then, sequentially:
1. Skeptic receives Optimist's Round N-1 summary → writes Round N rebuttal
2. Parent compresses Skeptic's Round N rebuttal
3. Optimist receives Skeptic's Round N summary → writes Round N rebuttal

This means the Optimist always rebuts the Skeptic's latest rebuttal, not stale Round 1 claims.

Each agent must explicitly evaluate: "Which of the other side's points are valid? Which do I concede?"

## Auto-Calibration

### Quick Mode
- 1 round only (parallel, no cross-examination)
- Parent writes synthesis directly from Round 1 outputs

### Detailed Mode
- Start with 3 rounds minimum
- After each round, check convergence:
  - If both agents concede >50% of the other's points → STOP (consensus reached)
  - If still divergent after 3 rounds → continue up to 2 more
  - Maximum: 5 rounds
- After final round, parent writes synthesis

## Parent Synthesis (Final)

After debate concludes, the parent writes:

### Agreement Zone
Points both agents agreed on. These are high-confidence findings.

### Disagreement Zone
Points of genuine disagreement. Present both perspectives fairly with the evidence each side cited.

### Verdict
- Overall assessment with confidence percentage (0-100%)
- Key takeaways (3-5 bullet points)
- Recommended next steps
- Risk level: Low / Medium / High / Critical

## Context Management

- Do NOT pass raw agent output between rounds (can be thousands of words)
- Always compress to structured bullet points between rounds
- Each agent only needs: the other side's key claims, evidence, and confidence
- Keep round summaries under 500 words each

## Agent Dispatch

Launch agents using the Agent tool:
- `research-skeptic` — model: sonnet, color: red, tools: Read, Grep
- `research-optimist` — model: sonnet, color: green, tools: Read, Grep

Pass the research findings in the agent prompt. Do not expect agents to search for information themselves.
