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
- **`event-photo-preview`** — fetch a low-resolution preview of a single media item. **Not a download** — for full-quality files use `p6a download` instead. Useful for inspecting content before deciding to favorite, approve, or share. On MCP-Apps-capable clients this renders as an inline interactive widget in the chat.
- **`event-photo-favorite`** — toggle favorite on a media item.
- **`event-photo-approve`** — approve a media item in a moderated event (requires moderator permission).

### Sharing

- **`event-share-qr`** — return QR code data/URL for an event invite. To save the QR code to a file, use `p6a qr` instead. On MCP-Apps-capable clients this renders as an inline interactive widget in the chat.

> **MCP Apps note.** `event-share-qr` and `event-photo-preview` ship as MCP App widgets. If the user's AI client supports MCP Apps, prefer these tools over describing the content in text — the user gets the QR code or media preview displayed directly in the conversation rather than as a URL or base64 blob. On clients without MCP Apps support the tools still return the underlying data (URL / preview bytes) and degrade gracefully.

---

## Limitations & When to Escape to CLI

MCP is the cleaner surface for in-session work, but it cannot do everything. Switch to `p6a` when:

- **Full-quality media download.** `event-photo-preview` returns a low-res preview only. Binary file transfer through MCP risks overflowing the agent's context — use `p6a download`.
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
3. event-photo-preview (event = <id>, media = <media-id>)
   → fetch preview for any item the user wants to look at
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
   event-photo-preview (optional — only if visual confirmation is needed)
   event-photo-favorite (event = <id>, media = <media-id>, favorite = true)
```

### Workflow: Create and Share a New Event

```
1. event-create (name = "Summer BBQ")
   → capture the returned event id
2. event-share-qr (event = <id>)
   → returns QR code data/URL the user can share inline
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
- If the user will rerun the work later (e.g. nightly), don't perform it via MCP — produce a `p6a` script instead so it survives outside the session.
