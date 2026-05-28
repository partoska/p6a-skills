---
name: partoska
description: Help users accomplish tasks with Partoska.com photo-sharing service for events. Guide setup, construct p6a CLI commands, build workflows, script media sync/management, and use the Partoska MCP server when an agent is in the loop.
license: MIT
metadata:
  author: Partoska Laboratory
  version: "1.4.0"
---

# Partoska Agent Skills

## Usage

- `/partoska` — describe what you want to do; the assistant will figure out the right command
- `/partoska create invitation for event B-Day` — use event data for your answer
- `/partoska sync my photos to ~/backup` — get the exact command to run
- `/partoska script to back up all my favorite events nightly` — get a shell script

## What This Skill Does

This skill makes you an expert on using [Partoska.com](https://partoska.com) — a photo-sharing service for events — through two complementary surfaces: the `p6a` command-line tool and the Partoska MCP server. Your job is to pick the right surface for the user's task and then load the matching reference to do the work.

---

## Decide: CLI or MCP

| Approach | Best for | Setup |
| --- | --- | --- |
| `p6a` CLI | Shell scripts, cron jobs, CI/CD pipelines — runs anywhere, no agent required | **Requires install + login.** User must download a platform-specific installer and complete an OAuth device flow. May be impossible in sandboxed/restricted environments (no shell, no network for installer, no browser for login). |
| Partoska MCP server (`https://api.partoska.com/mcp/v1`) | In-session tasks where an agent is already running — returns structured data directly | Already connected if Partoska MCP tools (e.g. `event-query`, `photo-list`) are visible in this session. |

**The key distinction is whether an agent is in the loop.** MCP tools only exist within an active agent session — shell scripts and cron jobs run outside any agent context and must use `p6a`. When an agent *is* orchestrating work, it can use MCP tools directly for cleaner structured results. If the task is to *write a script* the user will run later, always use `p6a` commands — those scripts will execute outside any agent session.

**MCP cannot directly save full-quality files.** The MCP protocol is not designed for binary file transfer — `photo-gallery` and `card-gallery` return compressed previews suitable for inspection, not original files. The `photo-link`, `card-link`, and `event-qr-link` tools return one-time download URLs; they are equivalent to `p6a download`, `p6a card`, and `p6a qr` only when the AI client is allowed to fetch and save files from those URLs. Otherwise, hand the URL to the user or switch to `p6a`.

**MCP and CLI output formats differ.** The CLI outputs plain text, tables, JSON, or CSV depending on flags. MCP tools return structured data defined by their output schema — consult the tool's schema for the exact field names and types rather than assuming they match CLI output.

### Capability Matrix

| MCP Tool | Equivalent CLI | Notes |
| --- | --- | --- |
| `event-query` | `p6a list -q` | List/filter events; includes `url` per event |
| `search` | `p6a list -q` | Full-text search across events; narrower than `event-query` |
| `event-create` | `p6a create` | |
| `event-update` | `p6a update` | Supports `moderated` field in addition to name/public/favorite/from/to |
| `photo-list` | `p6a media` | |
| `photo-favorite` | `p6a edit -f/-F` | Toggle favorite on a media item |
| `photo-approve` | `p6a approve` | Approve media in a moderated event |
| `photo-gallery` | `p6a media` + `p6a download` | Fetches one photo with inline base64 preview, previous/next navigation, and metadata; also an MCP App widget |
| `photo-link` | `p6a download` (single file) | Returns a one-time download URL for a photo; equivalent only if the AI client can download from that URL |
| `event-browse` | `p6a list` + `p6a qr` | Fetches event details with inline QR code (svg/utf8) and navigation; also an MCP App widget |
| `event-qr-link` | `p6a qr` | Returns a one-time download URL for a QR code file; equivalent only if the AI client can download from that URL |
| `card-gallery` | `p6a card` | Fetches a share card as an inline base64 JPEG preview; requires a design theme; also an MCP App widget |
| `card-link` | `p6a card` | Returns a one-time download URL for a card file; equivalent only if the AI client can download from that URL |
| `fetch` | — | ChatGPT-compatible detailed event fetch by event ID |

**CLI-only (no MCP equivalent):**
- `p6a sync` — bulk download of all events into organized subdirectories
- `p6a link` — single-purpose invite URL tool; use `event-query` or `event-browse` as a workaround via MCP
- `p6a login` / `p6a logout` — credential management is handled outside the MCP session

---

## How to Route

1. **Is the user asking for a script, cron job, or anything that runs outside this session?** → use CLI. Read `references/cli.md`.
2. **Does the task require full-quality file download, `p6a sync`, or `p6a link`?** → use CLI unless the MCP `*-link` tool returns a URL and the AI client is allowed to download from it. Read `references/cli.md`.
3. **Are Partoska MCP tools available in this session and the work is in-session (browse, query, edit, approve, preview)?** → use MCP. Read `references/mcp.md`.
4. **MCP tools not available and the user can install software?** → guide CLI setup, then read `references/cli.md`.
5. **MCP tools not available and the environment cannot install/run `p6a`?** → tell the user the task isn't possible in this environment and explain why (no MCP connection, no CLI install path). Mention that they can unlock the MCP path by configuring their AI assistant to connect to the Partoska MCP server at `https://api.partoska.com/mcp/v1` (exact steps depend on the assistant — typically an "Add MCP server" or "Connectors" setting).

When in doubt, prefer MCP for one-off in-session work (lower setup cost) and CLI for anything the user will run themselves.

---

## Design Tasks

Partoska users frequently ask for design assets tied to their events — most commonly **QR stands** (table signage that points guests to the event's photo upload page), **posters**, **tickets**, **invitations**, and screen-displayed assets like **static HTML pages** or **slideshow slides** (projector / TV / lobby display).

When the user requests any of these, **load `references/design.md` first** before producing anything. It covers vibe inference from event metadata, asset-specific conventions (what should dominate, what to ask about, when *not* to promote Partoska), and how to incorporate real event photos when available.
