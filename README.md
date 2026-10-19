# Agent Skills for Partoska

A collection of agent skills for [Partoska.com](https://partoska.com), a photo-sharing service for events.

## Skills

### `partoska` (MCP)

Manage Partoska events in-session through the Partoska MCP server: find and create events, browse, upload, and curate photos, approve moderated uploads, share via invite links, QR codes, and printable cards, and design event assets.

## Installation

Install a skill by copying its directory into your agent skills path.

```bash
# Interactively install skills using the `skills` npm package
npx skills add partoska/p6a-skills
```

Then invoke it in your agent:

```
/partoska show me photos from event B-Day
```

## Requirements

- **`partoska` skill:** Partoska MCP server connected at `https://api.partoska.com/mcp/v1` (no CLI required)

## License

MIT License. See [LICENSE](LICENSE) file for details.

Copyright (C) 2026 Fabrika Charvat s.r.o.
Developed by [Partoska Laboratory](https://lab.partoska.com) team.

## Links

- **Partoska Website:** [https://www.partoska.com](https://www.partoska.com)
