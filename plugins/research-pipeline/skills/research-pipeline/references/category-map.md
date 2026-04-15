# Category Map Reference

## Categories

| Category | Slug | Vault Path | Keywords |
|----------|------|-----------|----------|
| AI & Agents | `ai-agents` | `Research/AI-and-Agents/` | AI, LLM, GPT, Claude, agent, agentic, framework, model, neural, transformer, RAG, embedding, fine-tune, inference |
| Claude Code | `claude-code` | `Research/Claude-Code/` | Claude Code, claude-code, MCP, plugin, skill, hook, slash command, worktree, subagent, Anthropic CLI |
| Tools & Platforms | `tools-platforms` | `Research/Tools-and-Platforms/` | tool, platform, SaaS, app, builder, no-code, low-code, API, SDK, integration, release, launch, product |
| Automations & Workflows | `automations` | `Research/Automations-and-Workflows/` | automation, workflow, pipeline, n8n, Make, Zapier, MCP server, cron, webhook, trigger, orchestration |
| Business & GTM | `business-gtm` | `Research/Business-and-GTM/` | business, revenue, pricing, sales, consulting, GTM, go-to-market, client, agency, freelance, market, ROI |
| Content & Media | `content-media` | `Research/Content-and-Media/` | content, video, YouTube, podcast, ads, marketing, social media, creator, media, script, newsletter |
| PKM & Obsidian | `pkm-obsidian` | `Research/PKM-and-Obsidian/` | PKM, Obsidian, note-taking, knowledge management, vault, zettelkasten, second brain, linked notes |

## Classification Rules

1. **Keyword pre-filter:** Scan source content for keywords above. Assign categories with 3+ keyword matches.
2. **LLM classification:** For nuance — a source about "building an AI agent for sales automation" touches `ai-agents`, `automations`, and `business-gtm`.
3. **Max 3 categories** per research item. Pick the most relevant.
4. **Primary category** = where the markdown companion file is saved. Secondary/tertiary get cross-references in their overview.md files.
5. **Ambiguous cases:** Default to the broader category. An Obsidian plugin goes to `tools-platforms`, not `pkm-obsidian`, unless the article is specifically about PKM methodology.
