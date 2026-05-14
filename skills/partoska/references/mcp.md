# Partoska MCP Reference

Use this reference when the Partoska MCP server is connected (Partoska MCP tools such as `event-query` and `event-photo-list` are visible in the session) and the work happens entirely in this agent loop. See `../SKILL.md` for the CLI-vs-MCP decision and `cli.md` for tasks that need scripting or full-quality downloads.

---

## Available Tools

Tool names below are the canonical Partoska MCP tool names. Different MCP clients may surface them under host-specific prefixes (e.g. `partoska/event-query`, `mcp.partoska.event-query`) — match by the trailing tool name. Output schemas are defined by the server — consult each tool's schema for exact field names and types rather than assuming they match CLI output.

### Discovery

- **`event-query`** — list and filter events. Use this as the primary entry point when you need to find an event by attribute (owner, favorite, public, moderated, date range).
- **`search`** — full-text search across events. Narrower than `event-query`; reach for it when the user gives a free-form query like "Birthday 2024" without specifying which field to match.
- **`fetch`** — raw API fetch for endpoints not covered by other tools. Use sparingly; prefer dedicated tools when one exists.

### Event Lifecycle

- **`event-create`** — create a new event by name. Returns the populated event record.
- **`event-update`** — partially update an event. Send only the fields you intend to change (name, public, favorite, moderated, from/to). Mirrors `p6a update`.

### Media Inspection & Curation

- **`event-photo-list`** — list media items for a given event. Use to enumerate before previewing or curating.
- **`event-photo-gallery`** — interactive MCP App for browsing event photos. Use this when the user needs event context, favorite/count metadata, previous/next photo navigation, or an inline UI widget.
- **`event-photo-preview`** — no-UI tool that fetches one photo preview from an event as a base64-encoded JPEG image. Use this when the user explicitly asks for a single photo preview/image/data, the client needs a simple machine-readable image result, or no gallery browsing context is needed. **Not a download** — for full-quality files use `p6a download` instead (if `p6a` is available).
- **`event-photo-favorite`** — toggle favorite on a media item.
- **`event-photo-approve`** — approve a media item in a moderated event (requires moderator permission).

### Sharing

- **`event-browse`** — interactive MCP App for browsing an event. Use this when the user needs event details, favorite/count metadata, previous/next event navigation, or an inline UI widget.
- **`event-share-qr`** — no-UI tool that generates an event invite QR code payload. Use this when the user explicitly asks for a QR code/image/data, the client needs a simple machine-readable QR result, or no event browsing context is needed. For SVG and PNG formats the `data` field is base64-encoded; for UTF-8 it is a markdown code block. SVG and PNG may also return inline images for supported clients. To save the QR code to a file, use `p6a qr` (if `p6a` is available).

> **MCP Apps note.** `event-browse` and `event-photo-gallery` ship as MCP App widgets. Prefer them when the user wants to browse, compare, or navigate. Use `event-share-qr` and `event-photo-preview` for payload-only, no-UI requests where the user or client explicitly needs QR/image data.

---

## Limitations & When to Escape to CLI

MCP is the cleaner surface for in-session work, but it cannot do everything. Switch to `p6a` when:

- **Full-quality media download.** `event-photo-preview` and `event-photo-gallery` return low-res previews only. Binary file transfer through MCP risks overflowing the agent's context — use `p6a download`.
- **Bulk sync.** There is no MCP equivalent for `p6a sync` (organized subdirectory layout, idempotent skip-existing logic).
- **Invite link.** No dedicated tool. As a workaround, call `fetch` against the `GET /event/{id}/link` endpoint, or use `p6a link -e <event-id>`.
- **Login/logout.** Credentials are managed outside the MCP session. The MCP server is already authenticated for the active session — there is nothing to "log in" to via tools.
- **Producing a script for the user.** Scripts run outside any agent session, so they must use `p6a`. Even when MCP is available right now, switch to CLI commands when generating output the user will run later.

---

## Common Workflows

### Workflow: Find an Event and Inspect Its Media

```
1. event-query (filter: name contains "Birthday")
   → take the first matching event id
2. event-photo-list (event = <id>)
   → enumerate media items
3. event-photo-gallery (event = <id>, media = <initial-media-id>)
   → open an interactive gallery when the user wants to browse
   OR event-photo-preview (event = <id>, media = <media-id>)
   → fetch a no-UI preview payload for one explicit item
```

When the user says "show me photos from the Birthday event," resolve the event with `event-query` (or `search` for free-form input), then list and preview as needed. Don't dump every preview at once — fetch on demand to keep the context manageable.

### Workflow: Approve All Pending Media in a Moderated Event

```
1. event-photo-list (event = <id>)
   → filter the result for items where approved == false
2. event-photo-approve for each pending media id
```

This is the MCP equivalent of the CLI `jq | xargs` approval loop. Iterate the call rather than batching — there is no bulk approve tool.

### Workflow: Curate Favorites Across an Event

```
1. event-photo-list (event = <id>)
2. For each candidate the user wants flagged:
   event-photo-gallery (optional — when browsing/comparing photos helps)
   OR event-photo-preview (optional — when one no-UI preview is enough)
   event-photo-favorite (event = <id>, media = <media-id>, favorite = true)
```

### Workflow: Create and Share a New Event

```
1. event-create (name = "Summer BBQ")
   → capture the returned event id
2. event-browse (event = <id>)
   → opens the interactive event app when browsing/context is useful
   OR event-share-qr (event = <id>, format = png/svg/utf8)
   → returns a no-UI QR code payload when the user asks for QR data/image
```

If the user wants the QR as a file, switch to CLI: `p6a qr -e <event-id> -t invite.png`.

### Workflow: Hand Off to CLI for Download

```
1. event-query / search → resolve the event id in-session
2. Tell the user: run `p6a download -e <event-id> -t <target-dir>`
```

This is the canonical pattern when the user asks "download these photos" while MCP is connected — use MCP for discovery, then produce a CLI command for the actual file transfer.

---

## Tips

- Treat MCP responses as structured data; don't reformat them into CLI-style tables unless the user asks.
- When an MCP tool is missing for a task, check whether `fetch` covers the underlying endpoint before falling back to CLI.
- Event IDs are UUIDs and are interchangeable between MCP and CLI — an id obtained from `event-query` works in `p6a download -e <id>` without translation.
- If the user will rerun the work later (e.g. nightly), don't perform it via MCP — produce a `p6a` script instead so it survives outside the session (if `p6a` is available).
