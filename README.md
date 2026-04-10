# Agent Skills for Partoska Command-Line Tool

A collection of agent skills for the [`p6a`](https://github.com/partoska/p6a-cmd) command-line tool, the CLI for [Partoska.com](https://partoska.com) for a photo-sharing service for events.

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

- [`p6a`](https://github.com/partoska/p6a-cmd/releases) installed and authenticated (`p6a login`)

## License

MIT License. See [LICENSE](LICENSE) for details.

Copyright 2026 Fabrika Charvat s.r.o.

## Links

- **Partoska Website:** [https://www.partoska.com](https://www.partoska.com)
- **Project Website:** [https://lab.partoska.com/p6a](https://lab.partoska.com/p6a)
