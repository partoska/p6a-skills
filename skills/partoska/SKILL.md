---
name: partoska
description: Help users accomplish tasks with Partoska.com photo-sharing service for events. Guide setup, construct p6a CLI commands, build workflows, script media sync/management, and use the Partoska MCP server when an agent is in the loop.
license: MIT
metadata:
  author: Partoska Laboratory
  version: "1.0.0"
---

# Partoska Agent Skills

## Usage

- `/partoska` — describe what you want to do; the assistant will figure out the right command
- `/partoska create invitation for event B-Day` — use event data for your answer
- `/partoska sync my photos to ~/backup` — get the exact command to run
- `/partoska script to back up all my favorite events nightly` — get a shell script

## What This Skill Does

This skill makes you an expert on using [Partoska.com](https://partoska.com) — a photo-sharing service for events — via the `p6a` command-line tool and the Partoska MCP server. When invoked, you will:

1. Understand what the user is trying to accomplish
2. Determine the right approach — CLI or MCP (see below)
3. If using the CLI — verify the user has `p6a` installed and authenticated (guide setup if not)
4. Construct the exact commands or call the appropriate tools
5. Offer shell scripting help for automation workflows

---

## CLI vs MCP Server

Many actions can be performed two ways:

| Approach | Best for |
| --- | --- |
| `p6a` CLI | Shell scripts, cron jobs, CI/CD pipelines — runs anywhere, no agent required |
| Partoska MCP server (`https://api.partoska.com/mcp/v1`) | In-session tasks where an agent is already running — returns structured data directly |

**The key distinction is whether an agent is in the loop.** MCP tools only exist within an active agent session — shell scripts and cron jobs run outside any agent context and must use `p6a`. When an agent *is* orchestrating work, it can use MCP tools directly for cleaner structured results. However, if the task is to *write a script* the user will run later, always use `p6a` commands — those scripts will execute outside any agent session.

**MCP cannot download full-quality photos.** The MCP protocol is not designed for binary file transfer — `event-photo-preview` returns a low-resolution preview suitable for inspection, not the original file. Transferring full-resolution images through MCP would also risk overflowing the agent's context window. For any actual download of full-quality media, use `p6a download`.

**MCP and CLI output formats differ.** The CLI outputs plain text, tables, JSON, or CSV depending on flags. MCP tools return structured data defined by their output schema — consult the tool's schema for the exact field names and types rather than assuming they match CLI output.

### MCP Tools Available

When the Partoska MCP server is connected, the following tools are available to the agent:

| MCP Tool | Equivalent CLI | Notes |
| --- | --- | --- |
| `event-query` | `p6a list -q` | List/filter events |
| `search` | `p6a list -q` | Full-text search across events; narrower than `event-query` |
| `event-create` | `p6a create` | |
| `event-update` | `p6a update` | |
| `event-photo-list` | `p6a media` | |
| `event-photo-preview` | `p6a download` (single file) | Preview only in MCP, full quality in p6a |
| `event-share-qr` | `p6a qr` | Returns QR code data/URL; use `p6a qr` to save a file |
| `fetch` | — | Raw API fetch for endpoints not covered by other tools |

**CLI-only (no MCP equivalent):**
- `p6a sync` — bulk download of all events into organized subdirectories; has no MCP equivalent
- `p6a link` — fetch the primary invite URL for an event; use `fetch` as a workaround if needed
- `p6a login` / `p6a logout` — credential management is handled outside the MCP session

---

## Prerequisites (CLI only)

The following steps are only needed if using the `p6a` CLI. If the MCP server is available and sufficient for the task, skip this section.

### Step 1: Install p6a

Download the latest release for your platform from GitHub:

**[https://github.com/partoska/p6a-cmd/releases](https://github.com/partoska/p6a-cmd/releases)**

- **macOS**: download the `.pkg` installer and run it
- **Linux**: download the `.deb` (Debian/Ubuntu), `.rpm` (Fedora) installer and run it
- **Windows**: download the `.exe` installer (with `*_setup.exe` suffix) and run it

> **Windows note:** The shell examples in this skill use Bash syntax. On Windows, use PowerShell or `.bat` equivalents unless you are running inside MSYS2, Git Bash, Cygwin, or WSL — in those environments the Bash examples work as-is.

Verify the install:

```bash
p6a version
```

### Step 2: Log in

```bash
p6a login
```

This starts an OAuth device flow — follow the prompt to authorize in your browser. Credentials are saved to `~/.p6a/` and reused automatically. To use a custom config directory instead of `~/.p6a/`, pass `-D <path>` to any command (e.g. `p6a login -D ~/work/.p6a`).

Verify the connection:

```bash
p6a list
```

> If you see your events, you're ready to go. If `p6a` is not found, check that the binary is on your `PATH`.

---

## Command Reference

### Authentication

```bash
p6a login                          # Start OAuth device flow — opens a browser prompt
p6a login -i config.ini            # Import credentials from an existing INI file
p6a logout                         # Clear saved credentials
```

### List Events

```bash
p6a list                           # All accessible events (plain table)
p6a list -q "Birthday"             # Filter by name
p6a list -o                        # Only events you own
p6a list -f                        # Only favorited events
p6a list -F json                   # JSON output (pipe to jq)
p6a list -F csv                    # CSV output
p6a list -1                        # IDs only, one per line (pipe-friendly)
```

### Sync All Events

```bash
p6a sync -t ./photos               # Download all events into organized subdirs
p6a sync -t ~/backup -o            # Only events you own
p6a sync -t ~/backup -f            # Only favorited events
```

Sync creates one subdirectory per event, named `YYYY-MM-DD Event Name`, and skips files already downloaded. It is safe to run repeatedly.

### Create & Update Events

```bash
p6a create -n "Summer BBQ"                              # New event, print table row
p6a create -n "Summer BBQ" -1                           # New event, print ID only

p6a update -e <event-id> -n "New Name"                 # Rename event
p6a update -e <event-id> -p                            # Make public
p6a update -e <event-id> -P                            # Make private
p6a update -e <event-id> -f                            # Mark as favorite
p6a update -e <event-id> -F                            # Unmark favorite
p6a update -e <event-id> -S 2025-06-01T12:00:00        # Set start date/time
p6a update -e <event-id> -E 2025-06-01T18:00:00        # Set end date/time
```

Dates use ISO 8601 format. Multiple update flags can be combined in one call.

### Download Media

```bash
# Download all media for an event
p6a download -e <event-id> -t ./photos

# Download only your own uploads
p6a download -e <event-id> -t ./photos -o

# Download only favorited media
p6a download -e <event-id> -t ./photos -f

# Download a single media item by ID
p6a download -e <event-id> -m <media-id> -t ./output.jpg
```

Files are named `IMG_0001.jpg`, `MOV_0001.mp4`, etc., sequentially from existing files.

### List Media in an Event

```bash
p6a media -e <event-id>            # Plain table
p6a media -e <event-id> -F json    # JSON (pipe to jq)
p6a media -e <event-id> -1         # IDs only
p6a media -e <event-id> -o         # Only your uploads
p6a media -e <event-id> -f         # Only favorited items
```

### Share an Event

```bash
p6a link -e <event-id>             # Print invite URL to stdout
p6a qr -e <event-id>               # Download QR code as <id>-qr.png
p6a qr -e <event-id> -t invite.png # Download QR code to custom file
p6a qr -e <event-id> -s            # SVG format instead of PNG
```

---

## Common Workflows

### Workflow: Nightly Backup Script

```bash
#!/usr/bin/env bash
set -euo pipefail
p6a sync -t ~/backup
echo "Sync complete: $(date)"
```

Schedule with cron: `0 2 * * * /path/to/backup.sh`

### Workflow: Export Events to CSV

```bash
p6a list -F csv > events.csv
p6a list -q "Birthday" -F csv > birthday-events.csv   # filtered
```

### Workflow: Inspect Events with jq

```bash
# Pretty-print all events as JSON
p6a list -F json | jq .

# Extract just names and IDs
p6a list -F json | jq '.[] | {id: .id, name: .name}'

# Get IDs of all public events you own
p6a list -F json | jq -r '.[] | select(.public == true and .own == true) | .id'
```

### Workflow: Download Media from Multiple Events

```bash
p6a list -1 | xargs -I{} sh -c 'mkdir -p ./media/{} && p6a download -e {} -t ./media/{}'
```

### Workflow: Find and Download Media from an Event (PowerShell / Windows)

```powershell
# Find an event by name and download its media (no Bash required)
$eventId = p6a list -q "Birthday" -1 | Select-Object -First 1
p6a download -e $eventId -t .\photos
```

### Workflow: Find and Share an Event

```bash
# Get event ID by name, then print QR and invite link
EVENT_ID=$(p6a list -q "Birthday" -1 | head -1)
p6a qr -e "$EVENT_ID" -t birthday-qr.png
p6a link -e "$EVENT_ID"
```

---

## Key Flags Available on All Commands

| Flag               | Description                                       |
| ------------------ | ------------------------------------------------- |
| `-D, --dir <path>` | Use a custom config directory (default: `~/.p6a`) |
| `-h, --help`       | Show command help                                 |

---

## Tips

- Use `-1` / `--format one` to get IDs suitable for shell scripting.
- `p6a sync` is idempotent — safe to run repeatedly without re-downloading.
- Event IDs are UUIDs (e.g., `12cafe34-5b8a-4d2e-9f01-0203a4b5c6d7`); use `p6a list -1` to retrieve them.
- Use `-D` to maintain separate credential profiles for different Partoska accounts.
