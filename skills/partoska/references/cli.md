# p6a CLI Reference

Use this reference when the task involves the `p6a` command-line tool — shell scripts, cron jobs, CI/CD, or any in-session work where the MCP server is unavailable. See `../SKILL.md` for the CLI-vs-MCP decision.

---

## Prerequisites

The CLI requires installing the binary and completing a one-time OAuth login. If the user's environment cannot install software or open a browser, the CLI path is not viable — fall back to MCP or tell the user the task isn't possible.

### Step 1: Install p6a

#### Option A: npm (simplest — no platform-specific installer needed)

```bash
npm install -g @partoska/p6a
```

Or run without installing (useful for one-off use or environments where global installs are restricted):

```bash
npx @partoska/p6a login
npx @partoska/p6a <command>
```

Prefer this option when the user has Node.js available — it works on macOS, Linux, and Windows without choosing a platform-specific asset.

#### Option B: Platform-specific binary from GitHub Releases

Download the latest release for the user's platform from GitHub:

**[https://github.com/partoska/p6a-cmd/releases](https://github.com/partoska/p6a-cmd/releases)**

If network access is available and the user wants exact commands, resolve the latest release via the GitHub Releases API instead of inventing versioned URLs:

```bash
curl -L https://api.github.com/repos/partoska/p6a-cmd/releases/latest
```

Select the installer from `assets[].name`, use its `browser_download_url`, and verify against its `digest` when possible. Do **not** hard-code a version or construct a release URL by guessing.

#### Choose the right asset

| Platform | Preferred interactive install | Portable/headless fallback |
| --- | --- | --- |
| macOS | `p6a_<version>_darwin_universal.pkg` | `p6a_<version>_darwin_universal.tar.gz` or bare `p6a_<version>_darwin_universal` |
| Debian/Ubuntu amd64 | `p6a_<version>_linux_amd64.deb` | `p6a_<version>_linux_amd64.tar.gz` or bare `p6a_<version>_linux_amd64` |
| Debian/Ubuntu arm64 | `p6a_<version>_linux_arm64.deb` | `p6a_<version>_linux_arm64.tar.gz` or bare `p6a_<version>_linux_arm64` |
| Fedora/RHEL amd64 | `p6a_<version>_linux_amd64.rpm` | `p6a_<version>_linux_amd64.tar.gz` or bare `p6a_<version>_linux_amd64` |
| Fedora/RHEL arm64 | `p6a_<version>_linux_arm64.rpm` | `p6a_<version>_linux_arm64.tar.gz` or bare `p6a_<version>_linux_arm64` |
| Windows amd64 | `p6a_<version>_windows_amd64_setup.exe` | `.msi`, `.zip`, or bare `p6a_<version>_windows_amd64.exe` |
| Windows arm64 | `p6a_<version>_windows_arm64_setup.exe` | `.msi`, `.zip`, or bare `p6a_<version>_windows_arm64.exe` |

Use `uname -s` for OS and `uname -m` for CPU architecture. Map `x86_64` to `amd64`; map `aarch64` or `arm64` to `arm64`. On Linux, inspect `/etc/os-release` to choose `.deb` for Debian/Ubuntu-family systems and `.rpm` for Fedora/RHEL-family systems.

#### Install examples

Window package:

```bat
p6a_<version>_windows_amd64_setup.exe
```

macOS package:

```bash
sudo installer -pkg ./p6a_<version>_darwin_universal.pkg -target /
```

Debian/Ubuntu:

```bash
sudo apt install ./p6a_<version>_linux_amd64.deb
```

Fedora/RHEL:

```bash
sudo dnf install ./p6a_<version>_linux_amd64.rpm
```

Portable Linux/macOS tarball:

```bash
tar -xzf ./p6a_<version>_<platform>.tar.gz
chmod +x ./p6a
mkdir -p ~/.local/bin
mv ./p6a ~/.local/bin/p6a
```

Portable bare binary:

```bash
chmod +x ./p6a_<version>_<platform>
mkdir -p ~/.local/bin
mv ./p6a_<version>_<platform> ~/.local/bin/p6a
```

Make sure `~/.local/bin` is on `PATH` when using portable installs.

#### Verify downloaded assets

When the release API provides a `digest` such as `sha256:<hex>`, verify the downloaded file before installing:

```bash
shasum -a 256 ./p6a_<version>_<platform>.<ext>
```

Compare the output hash to the asset digest from the release API.

> **Windows note:** The shell examples in this reference use Bash syntax. On Windows, use PowerShell or `.bat` equivalents unless you are running inside MSYS2, Git Bash, Cygwin, or WSL — in those environments the Bash examples work as-is.

Verify the install:

```bash
p6a version
```

### Step 2: Log in

```bash
p6a login
```

This starts an OAuth device flow — follow the prompt to authorize in a browser. Credentials are saved to `~/.p6a/` and reused automatically. To use a custom config directory instead of `~/.p6a/`, pass `-D <path>` to any command (e.g. `p6a login -D ~/work/.p6a`).

Verify the connection:

```bash
p6a list
```

> If events are listed, the setup is complete. If `p6a` is not found, check that the binary is on `PATH`.

### Install Troubleshooting

- `p6a: command not found`: open a new terminal, verify the installer updated `PATH`, or use the full path to the binary.
- Portable install still not found: add the install directory (for example `~/.local/bin`) to `PATH`.
- macOS blocks the binary or package: prefer the `.pkg` installer; if macOS still blocks execution, use the Security & Privacy prompt in System Settings to allow it.
- Linux package install fails for permissions: run the package manager command with `sudo`, or use the portable tarball in a user-owned directory.
- Linux package format is wrong: use `.deb` for Debian/Ubuntu-family systems and `.rpm` for Fedora/RHEL-family systems.
- Windows terminal cannot find `p6a`: close and reopen the terminal after install; if needed, use the full install path or rerun the setup installer.

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
p6a card -e <event-id> -d gold                   # Luxury/elegant theme (PDF)
p6a card -e <event-id> -d romantic               # Romantic/wedding theme (PDF)
p6a card -e <event-id> -d neon                   # Party/nightlife theme (PDF)

p6a card -e <event-id> -d bday -F jpg            # JPEG instead of PDF
p6a card -e <event-id> -d bday -t card.pdf       # Custom output file
p6a card -e <event-id> -d tech -b                # White background (no-background mode)
p6a card -e <event-id> -d bday -l cs -p A4       # Czech locale, A4 paper
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
