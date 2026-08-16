---
name: p6a
description: Construct p6a CLI commands and workflows for Partoska.com photo-sharing — install and login, list/create/update events, upload and moderate media, download photos in full quality, bulk sync, QR codes and share cards, shell scripts and cron jobs, and design event assets.
license: MIT
metadata:
  author: Partoska Laboratory
  version: "2.1.1"
---

# p6a CLI Skill

## Usage

- `/p6a` — describe what you want to do; the assistant will figure out the right command
- `/p6a sync my photos to ~/backup` — get the exact command to run
- `/p6a script to back up all my favorite events nightly` — get a shell script
- `/p6a make a QR stand for event B-Day` — design an asset using event data

## What This Skill Does

This skill makes you an expert on the `p6a` command-line tool for [Partoska.com](https://partoska.com) — a photo-sharing service for events. `p6a` runs anywhere a shell runs — interactive terminals, shell scripts, cron jobs, CI/CD pipelines — and covers authentication, event management, media upload/download/moderation, full-quality bulk sync, and sharing via invite links, QR codes, and printable cards.

## Setup

`p6a` requires a one-time install and OAuth login. When the user asks how to install, download, update, verify, or log in — or a command fails with `p6a: command not found` or an auth error — read `references/install.md`; it covers npm and platform-specific installers, release-asset selection, checksums, the login flow, and troubleshooting. If the user's environment cannot install software or open a browser for login, the task isn't possible with `p6a` in that environment — say so rather than improvising.

Quick sanity check on an already-set-up machine:

```bash
p6a version   # binary installed and on PATH
p6a list      # credentials valid — prints accessible events
```

## Design Tasks

Partoska users frequently ask for design assets tied to their events — **QR stands**, **posters**, **tickets**, **invitations**, **static HTML pages**, **slideshow slides**, and photo-driven pieces like **collages** or **photo recaps**. When the user requests any of these, **load `references/design.md` first** before producing anything. It covers vibe inference from event metadata, asset-specific conventions (what should dominate, what to ask about, when *not* to promote Partoska), and how to incorporate full-quality event photos via `p6a download`.

---

## Command Reference

> **Windows note:** The shell examples below use Bash syntax. On Windows, use PowerShell or `.bat` equivalents unless you are running inside MSYS2, Git Bash, Cygwin, or WSL — in those environments the Bash examples work as-is.

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
p6a update -e <event-id> -m                            # Enable upload moderation
p6a update -e <event-id> -M                            # Disable upload moderation
p6a update -e <event-id> -S 2025-06-01T12:00:00        # Set start date/time
p6a update -e <event-id> -E 2025-06-01T18:00:00        # Set end date/time
```

Dates use ISO 8601 format. Multiple update flags can be combined in one call.

### Edit & Approve Media

```bash
# Favorite / unfavorite a media item
p6a edit -e <event-id> -m <media-id> -f    # mark as favorite
p6a edit -e <event-id> -m <media-id> -F    # unmark favorite

# Approve a media item in a moderated event (requires moderator permission)
p6a approve -e <event-id> -m <media-id>
```

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

### Upload a Photo

Upload a media file to an event. Prints the returned media UUID on success.

```bash
p6a upload -e <event-id> -s photo.jpg
```

On moderated events, uploaded media are created as unapproved and won't be visible to other guests until a moderator runs `p6a approve`.

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
p6a link -e <event-id>                           # Print invite URL to stdout
p6a qr -e <event-id>                             # Download QR code as <id>-qr.png
p6a qr -e <event-id> -t invite.png               # Download QR code to custom file
p6a qr -e <event-id> -F svg                      # SVG format instead of PNG
```

### Download a Share Card

Pre-built themed cards containing the event QR code, intended to prompt guests to upload photos. Not a general-purpose design tool — for custom assets see `references/design.md`.

```bash
p6a card -e <event-id> -d bday                   # Birthday/anniversary theme (PDF)
p6a card -e <event-id> -d tech                   # Corporate/conference theme (PDF)
p6a card -e <event-id> -d match                  # Sport/tournament theme (PDF)
p6a card -e <event-id> -d forest                 # Outdoor/nature theme (PDF)
p6a card -e <event-id> -d garden                 # Garden/floral theme (PDF)
p6a card -e <event-id> -d gold                   # Luxury/elegant theme (PDF)
p6a card -e <event-id> -d romantic               # Romantic/wedding theme (PDF)
p6a card -e <event-id> -d silver                 # Cool/modern elegant theme (PDF)
p6a card -e <event-id> -d neon                   # Party/nightlife theme (PDF)
p6a card -e <event-id> -d nineties               # Retro/1990s theme (PDF)
p6a card -e <event-id> -d cottage                # Cottage getaways, cabin weekends, countryside retreats (PDF)

p6a card -e <event-id> -d bday -F jpg            # JPEG instead of PDF
p6a card -e <event-id> -d bday -t card.pdf       # Custom output file
p6a card -e <event-id> -d tech -b                # White background (no-background mode)
p6a card -e <event-id> -d forest -l cs -L business -p letter  # Czech, tiled layout
```

Supported card options are locale `en` (default), `cs`, `sk`, `pl`, `ru`, or `es`, layout `single` (default) or `business`, and paper size `a4` (default) or `letter`. Downloading a share card requires organizer (admin) access.

### Global Help and Configuration

| Invocation                       | Description                                                                 |
| -------------------------------- | --------------------------------------------------------------------------- |
| `-D, --dir <path>`               | Use a custom config directory on supported commands (default: `~/.p6a`)     |
| `p6a help`, `p6a -h`, `p6a --help` | Show top-level command help                                                |

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
p6a list -F json | jq -r '.[] | select(.public == true and .owner == true) | .id'

# Get IDs of moderated events (uploads need approval before becoming visible)
p6a list -F json | jq -r '.[] | select(.moderated == true) | .id'

# List unapproved media items for an event
p6a media -e <event-id> -F json | jq '.[] | select(.approved == false)'
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

### Workflow: Upload Multiple Photos

```bash
EID=12cafe34-5b8a-4d2e-9f01-0203a4b5c6d7
for f in *.jpg; do
  p6a upload -e "$EID" -s "$f"
done
```

### Workflow: Approve All Pending Media in a Moderated Event

```bash
EID=12cafe34-5b8a-4d2e-9f01-0203a4b5c6d7
p6a media -e "$EID" -F json | \
  jq -r '.[] | select(.approved == false) | .id' | \
  xargs -I{} p6a approve -e "$EID" -m {}
```

### Workflow: Find and Share an Event

```bash
# Get event ID by name, then print QR and invite link
EVENT_ID=$(p6a list -q "Birthday" -1 | head -1)
p6a qr -e "$EVENT_ID" -t birthday-qr.png
p6a link -e "$EVENT_ID"
```

---

## Tips

- Use `-1` / `--format one` to get IDs suitable for shell scripting.
- `p6a sync` is idempotent — safe to run repeatedly without re-downloading.
- Event IDs are UUIDs (e.g., `12cafe34-5b8a-4d2e-9f01-0203a4b5c6d7`); use `p6a list -1` to retrieve them.
- Use `-D` to maintain separate credential profiles for different Partoska accounts.
