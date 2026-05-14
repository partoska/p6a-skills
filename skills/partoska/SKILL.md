---
name: partoska
description: Help users accomplish tasks with Partoska.com photo-sharing service for events. Guide setup, construct p6a CLI commands, build workflows, script media sync/management, and use the Partoska MCP server when an agent is in the loop.
license: MIT
metadata:
  author: Partoska Laboratory
  version: "1.2.0"
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
| Partoska MCP server (`https://api.partoska.com/mcp/v1`) | In-session tasks where an agent is already running — returns structured data directly | Already connected if Partoska MCP tools (e.g. `event-query`, `event-photo-list`) are visible in this session. |

**The key distinction is whether an agent is in the loop.** MCP tools only exist within an active agent session — shell scripts and cron jobs run outside any agent context and must use `p6a`. When an agent *is* orchestrating work, it can use MCP tools directly for cleaner structured results. If the task is to *write a script* the user will run later, always use `p6a` commands — those scripts will execute outside any agent session.

**MCP cannot download full-quality photos.** The MCP protocol is not designed for binary file transfer — `event-photo-preview` and `event-photo-gallery` return low-resolution previews suitable for inspection, not the original file. Transferring full-resolution images through MCP would also risk overflowing the agent's context window. For any actual download of full-quality media, use `p6a download` (if `p6a` is available).

**MCP and CLI output formats differ.** The CLI outputs plain text, tables, JSON, or CSV depending on flags. MCP tools return structured data defined by their output schema — consult the tool's schema for the exact field names and types rather than assuming they match CLI output.

### Capability Matrix

| MCP Tool | Equivalent CLI | Notes |
| --- | --- | --- |
| `event-query` | `p6a list -q` | List/filter events |
| `search` | `p6a list -q` | Full-text search across events; narrower than `event-query` |
| `event-create` | `p6a create` | |
| `event-update` | `p6a update` | |
| `event-photo-list` | `p6a media` | |
| `event-photo-favorite` | `p6a edit -f/-F` | Toggle favorite on a media item |
| `event-photo-approve` | `p6a approve` | Approve media in a moderated event |
| `event-photo-gallery` | `p6a media` + `p6a download` | Interactive MCP App for browsing photos with event context, counts, favorite metadata, previews, and previous/next navigation |
| `event-photo-preview` | `p6a download` (single file) | No-UI single photo preview payload; preview only in MCP, full quality in p6a |
| `event-browse` | `p6a list` + `p6a qr` | Interactive MCP App for event details, metadata, navigation, and invite browsing |
| `event-share-qr` | `p6a qr` | No-UI QR payload tool; use when the user explicitly asks for QR image/data |
| `fetch` | — | Raw API fetch for endpoints not covered by other tools |

**CLI-only (no MCP equivalent):**
- `p6a sync` — bulk download of all events into organized subdirectories
- `p6a link` — fetch the primary invite URL for an event; use `fetch` as a workaround if needed
- `p6a login` / `p6a logout` — credential management is handled outside the MCP session

---

## How to Route

1. **Is the user asking for a script, cron job, or anything that runs outside this session?** → use CLI. Read `references/cli.md`.
2. **Does the task require full-quality media download or `p6a sync`/`p6a link`?** → use CLI. Read `references/cli.md`.
3. **Are Partoska MCP tools available in this session and the work is in-session (browse, query, edit, approve, preview)?** → use MCP. Read `references/mcp.md`.
4. **MCP tools not available and the user can install software?** → guide CLI setup, then read `references/cli.md`.
5. **MCP tools not available and the environment cannot install/run `p6a`?** → tell the user the task isn't possible in this environment and explain why (no MCP connection, no CLI install path). Mention that they can unlock the MCP path by configuring their AI assistant to connect to the Partoska MCP server at `https://api.partoska.com/mcp/v1` (exact steps depend on the assistant — typically an "Add MCP server" or "Connectors" setting).

When in doubt, prefer MCP for one-off in-session work (lower setup cost) and CLI for anything the user will run themselves.

---

## Design Tasks

Partoska users frequently ask for design assets tied to their events — most commonly **QR stands** (table signage that points guests to the event's photo upload page), **posters**, **tickets**, **invitations**, and screen-displayed assets like **static HTML pages** or **slideshow slides** (projector / TV / lobby display).

When the user requests any of these, **load `references/design.md` first** before producing anything. It covers vibe inference from event metadata, asset-specific conventions (what should dominate, what to ask about, when *not* to promote Partoska), and how to incorporate real event photos when available.
