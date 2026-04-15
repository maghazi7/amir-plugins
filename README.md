# Claude Code Plugins

A marketplace of plugins for [Claude Code](https://claude.com/claude-code).

## Plugins

### `research-pipeline` — v1.0.0

Research any topic with web search, content extraction, an adversarial Skeptic-vs-Optimist debate, and Obsidian vault integration. Produces a polished HTML report plus a markdown companion routed into your vault.

```
/research <topic>
/research <url> [<url> ...]
/research <topic> --quick   # 1 debate round instead of 3-5
```

See [`plugins/research-pipeline/README.md`](plugins/research-pipeline/README.md) for full docs.

## Install

```bash
# add this marketplace
/plugin marketplace add maghazi7/claude-plugins

# install the plugin
/plugin install research-pipeline@claude-plugins
```

## Configuration

Before first use, set your vault root (where your Obsidian `Research/` folder lives):

```bash
export RESEARCH_VAULT_ROOT="$HOME/my-vault"
```

Add that line to your `~/.zshrc` or `~/.bashrc` to persist it. If the variable is unset, the plugin defaults to `~/Research/`. See [`plugins/research-pipeline/README.md`](plugins/research-pipeline/README.md) for the full vault structure that will be created.

## License

MIT — see [`LICENSE`](LICENSE).
