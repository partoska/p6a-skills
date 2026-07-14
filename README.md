# Agent Skills for Partoska

A collection of agent skills for [Partoska.com](https://partoska.com), a photo-sharing service for events.

## Skills

### `partoska` (MCP)

Manage Partoska events in-session through the Partoska MCP server: find and create events, browse, upload, and curate photos, approve moderated uploads, share via invite links, QR codes, and printable cards, and design event assets.

### `p6a` (CLI)

Construct commands and workflows for the Partoska's [`p6a`](https://github.com/partoska/p6a-cmd) command-line tool: install and login, event management, upload and moderation, full-quality downloads and bulk sync, QR codes and share cards, shell scripts and cron jobs.

Use `partoska` for interactive, in-session work through an MCP-connected assistant; use `p6a` for anything that runs in a shell, such as scripts, cron jobs, and bulk backups.

## Installation

Install a skill by copying its directory into your agent skills path.

```bash
# Interactively install skills using the `skills` npm package
npx skills add partoska/p6a-skills
```

Then invoke it in your agent:

```
/partoska show me photos from event B-Day
/p6a sync my photos to ~/backup
```

## Requirements

- **`partoska` skill:** Partoska MCP server connected at `https://api.partoska.com/mcp/v1` (no CLI required)
- **`p6a` skill:** [`p6a`](https://github.com/partoska/p6a-cmd/releases) installed and authenticated (`p6a login`); the skill itself can guide you through both

## License

MIT License. See [LICENSE](LICENSE) file for details.

Copyright (C) 2026 Fabrika Charvat s.r.o.
Developed by [Partoska Laboratory](https://lab.partoska.com) team.

## Links

- **Partoska Website:** [https://www.partoska.com](https://www.partoska.com)
- **Project Website:** [https://www.partoska.com/p6a](https://www.partoska.com/p6a)
- **Partoska Command-Line Tool:** [https://github.com/partoska/p6a-cmd](https://github.com/partoska/p6a-cmd)
