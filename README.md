# Agent Skills for Partoska

A collection of agent skills for [Partoska.com](https://partoska.com), a photo-sharing service for events.

## Skills

### `partoska` (MCP)

Manage Partoska events in-session through the Partoska MCP server: find and create events, browse, upload, and curate photos, approve moderated uploads or revoke an approval, share via invite links, QR codes, and printable cards, and design event assets.

## Installation

### Claude Code plugin

Installs the skill and connects the Partoska MCP server.

```bash
claude plugin marketplace add partoska/p6a-skills
claude plugin install partoska@partoska
```

### Codex plugin

Installs the skill and connects the Partoska MCP server.

```bash
codex plugin marketplace add partoska/p6a-skills
codex plugin add partoska@partoska
```

### Skill only

Install a skill by copying its directory into your agent skills path.

```bash
# Interactively install skills using the `skills` npm package
npx skills add partoska/p6a-skills
```

If you already use the Partoska connector on claude.ai or ChatGPT, install the skill only. The plugins add their own connection to the same server, so the tools would show up twice.

Then invoke it in your agent:

```
/partoska show me photos from event B-Day
```

## Requirements

- **`partoska` skill:** Partoska MCP server connected at `https://api.partoska.com/mcp/v1` (no CLI required). The plugins connect it for you.

## Repository Layout

The repository is a plugin marketplace for Claude Code (`.claude-plugin/marketplace.json`) and Codex (`.agents/plugins/marketplace.json`). Each plugin lives in `plugins/<name>/` and holds its skills under `plugins/<name>/skills/`. `skills/<name>` at the root is a symlink to the plugin's copy, so existing paths keep working. Codex does not follow symlinks, so the real files must stay under `plugins/`.

## License

MIT License. See [LICENSE](LICENSE) file for details.

Copyright (C) 2026 Fabrika Charvat s.r.o.
Developed by [Partoska Laboratory](https://lab.partoska.com) team.

## Links

- **Partoska Website:** [https://www.partoska.com](https://www.partoska.com)
- **Privacy Policy:** [https://www.partoska.com/legal/privacy-policy](https://www.partoska.com/legal/privacy-policy)
- **Terms of Service:** [https://www.partoska.com/legal/terms-of-service](https://www.partoska.com/legal/terms-of-service)
