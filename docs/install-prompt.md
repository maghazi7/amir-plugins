You are helping install the `research-pipeline` plugin into Claude Code. This plugin adds a research workflow: web search → adversarial Skeptic/Optimist debate → polished HTML report → Obsidian vault integration. Your job is to walk through the install, check for dependencies, and confirm everything works.

---

## Step 0 — Confirm Claude Code is running

You are already running inside Claude Code, so this is confirmed. Good.

---

## Step 1 — Detect OS and shell

Run this and report what you find:

```bash
uname -s && echo $SHELL
```

- **macOS or Linux**: proceed with the standard bash flow below.
- **Windows**: recommend WSL2 first (full Linux environment, easiest path). If the user wants to use PowerShell natively, switch `export VAR=value` to `$env:VAR = "value"` and `which` to `Get-Command` throughout. Git Bash is a lighter middle ground if WSL2 isn't available.

---

## Step 2 — Add the marketplace and install the plugin

Ask the user to run these two commands (they're Claude Code slash commands, not shell commands — paste them into the Claude Code prompt):

```
/plugin marketplace add maghazi7/amir-plugins
/plugin install research-pipeline@amir-plugins
```

Also install the required companion plugin from the official marketplace:

```
/plugin install frontend-design@claude-plugins-official
```

Wait for the user to confirm both installed successfully before continuing.

---

## Step 3 — Set RESEARCH_VAULT_ROOT

This env var tells the plugin where your Obsidian vault lives. It defaults to `~/Research/` but you should set it explicitly.

Ask the user: **Where is your Obsidian vault?** (The folder that contains your `.obsidian/` directory.)

Then add it to their shell rc file. **Before running, confirm with the user** — this modifies their shell config:

**macOS/Linux (bash):**
```bash
echo 'export RESEARCH_VAULT_ROOT="$HOME/path/to/your/vault"' >> ~/.bashrc
source ~/.bashrc
```

**macOS (zsh):**
```bash
echo 'export RESEARCH_VAULT_ROOT="$HOME/path/to/your/vault"' >> ~/.zshrc
source ~/.zshrc
```

Replace the path with what the user told you. Verify it stuck:
```bash
echo $RESEARCH_VAULT_ROOT
```

---

## Step 4 — Install runtime dependencies

Check and install each dependency. Explain what each does before installing — these are system-wide changes, so confirm with the user first.

### Playwright CLI (primary content extraction)

Check if installed:
```bash
which playwright-cli || npx playwright-cli --version 2>/dev/null
```

If missing, explain: *"This is used to fetch web content from JavaScript-heavy sites. It's an npm package installed globally."* Then confirm and install:
```bash
npm install -g @anthropic-ai/playwright-cli
```

### defuddle (article extraction)

No install needed — it runs via `npx`. Verify it's accessible:
```bash
npx defuddle --version
```

If this fails, `npx` itself may be missing (node/npm not installed). Fix node first.

### youtube-transcript-api (optional — only needed for YouTube research)

Ask: *"Will you research YouTube videos? If not, skip this."*

If yes, check:
```bash
pip3 show youtube-transcript-api 2>/dev/null | grep Name
```

If missing: *"This Python package pulls transcripts from YouTube videos."* Confirm, then install:
```bash
pip3 install youtube-transcript-api
```

---

## Step 5 — Verify everything works

Run the built-in dependency check:

```
/research hello world --quick
```

This triggers Phase 0, which checks every dependency and reports what's missing. Read the output and fix anything flagged before moving on.

---

## You're ready

Once the verify step passes cleanly, you can run your first real research request:

```
/research <your topic here>
```

The plugin will search the web, extract content, run a Skeptic vs. Optimist debate on the findings, and produce a formatted HTML report saved to your vault's `Research/reports/` folder.

If anything breaks during install, share the error output and we'll debug it together.
