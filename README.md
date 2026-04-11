# Agent Skills for Partoska

A collection of agent skills for [Partoska.com](https://partoska.com), a photo-sharing service for events. Supports both the [`p6a`](https://github.com/partoska/p6a-cmd) command-line tool and the Partoska MCP server.

## Installation

Install a skill by copying its directory into your agent skills path.

```bash
# Interactively install skills using the `skills` npm package
npx skills add https://github.com/partoska/p6a-skills
```

Then invoke it in your agent:

```
/partoska sync my photos to ~/backup
```

## Requirements

- **CLI usage:** [`p6a`](https://github.com/partoska/p6a-cmd/releases) installed and authenticated (`p6a login`)
- **MCP usage:** Partoska MCP server connected at `https://api.partoska.com/mcp/v1` (no CLI required)

## License

MIT License. See [LICENSE](LICENSE) file for details.

Copyright (C) 2026 Fabrika Charvat s.r.o.
Developed by [Partoska Laboratory](https://lab.partoska.com) team.

## Links

- **Partoska Website:** [https://www.partoska.com](https://www.partoska.com)
- **Project Website:** [https://lab.partoska.com/p6a](https://lab.partoska.com/p6a)
- **Partoska Command-Line Tool:** [https://github.com/partoska/p6a-cmd](https://github.com/partoska/p6a-cmd)
