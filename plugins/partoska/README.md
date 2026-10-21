# Partoska Plugin

Manage [Partoska.com](https://www.partoska.com) photo-sharing events from Claude Code or Codex. Partoska collects photos and videos from guests at weddings, birthdays, parties, and other events through an invite link or QR code.

## What It Does

- Find, create, and update events: name, dates, and visibility.
- Browse, preview, upload, and favorite photos.
- Moderate uploads: approve pending photos, put them on hold, or revoke an approval.
- Review photos on your behalf: check uploads against your rules, pick favorites, run a privacy check before an event goes public, or find a specific moment.
- Help hosts during a live event with coverage checks, guest nudges, and live moderation.
- Share events through invite links, QR codes, and printable cards.
- Design event assets such as QR stands, posters, invitations, photo collages, and recap stories.

## Components

- **Skill `partoska`:** instructions that teach your agent how to use the Partoska tools and when to load each reference guide.
- **MCP server `partoska`:** a remote HTTP connection to `https://api.partoska.com/mcp/v1`.

## Data and Connections

The plugin connects only to the Partoska MCP server at `https://api.partoska.com/mcp/v1`. Event details, photos, and moderation actions are sent to and read from Partoska through that server. The plugin runs no local code, hooks, or scripts, and it sends no data to any other service.

## Installation

### Claude Code

In Claude Code, run:

```
/plugin install partoska --marketplace partoska/p6a-skills
```

Or from your shell:

```bash
claude plugin marketplace add partoska/p6a-skills
claude plugin install partoska@partoska
```

### Codex

```bash
codex plugin marketplace add partoska/p6a-skills
codex plugin add partoska@partoska
```

If you already use the Partoska connector on claude.ai or ChatGPT, the plugin adds a second connection to the same server, so the tools would show up twice.

## Usage

Once the plugin is installed, ask your agent about your events:

```
Show me the latest photos from my event B-Day
Create an event for Saturday and give me the QR code
Moderate the pending photos at my party
```

## Links

- **Website:** [https://www.partoska.com](https://www.partoska.com)
- **Support:** [https://www.partoska.com/contact](https://www.partoska.com/contact)
- **Privacy Policy:** [https://www.partoska.com/legal/privacy-policy](https://www.partoska.com/legal/privacy-policy)
- **Terms of Service:** [https://www.partoska.com/legal/terms-of-service](https://www.partoska.com/legal/terms-of-service)

## License

MIT License. Copyright (C) 2026 Fabrika Charvat s.r.o. Developed by [Partoska Laboratory](https://lab.partoska.com) team.
