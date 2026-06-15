# Partoska MCP Reference

Use this reference when the Partoska MCP server is connected (Partoska MCP tools such as `event-query` and `photo-list` are visible in the session) and the work happens entirely in this agent loop. See `../SKILL.md` for the CLI-vs-MCP decision and `cli.md` for tasks that need scripting or full-quality downloads.

---

## Available Tools

Tool names below are the canonical Partoska MCP tool names. Different MCP clients may surface them under host-specific prefixes (e.g. `partoska/event-query`, `mcp.partoska.event-query`) — match by the trailing tool name. Output schemas are defined by the server — consult each tool's schema for exact field names and types rather than assuming they match CLI output.

### Discovery

- **`event-query`** — search and list events with rich metadata. Use this as the primary entry point when you need to find an event by name, sort results, limit result count, or inspect event attributes such as admin/favorite/public/moderated state and guest/photo counts. Returns `url` per event — use it to retrieve the invite URL without a separate tool call.
- **`search`** — full-text search across events. Narrower than `event-query`; reach for it when the user gives a free-form query like "Birthday 2024" without specifying which field to match.
- **`fetch`** — ChatGPT-compatible detailed event fetch by event ID. Use it when a client expects the simplified `id`/`title`/`text`/`url` result shape; prefer `event-query` or `event-browse` for richer structured data.

### Event Lifecycle

- **`event-create`** — create a new event by name. Returns the populated event record.
- **`event-update`** — partially update an event. Send only the fields you intend to change (`name`, `public`, `favorite`, `moderated`, `from`/`to`).

### Photo Inspection & Curation

- **`photo-list`** — list photos for a given event. Use to enumerate before previewing or curating. Check `isApproved` to identify pending photos in a moderated event.
- **`photo-gallery`** — fetch a single photo with full metadata: base64-encoded preview image (`data`), `previous`/`next` photo IDs for navigation, and favorite state and count. Also ships as an MCP App widget when the client supports it. Use this when the user needs to see a photo inline, navigate the gallery, or view photo metadata — it covers both interactive and data-only use cases. For a full-quality download link, use `photo-link`.
- **`photo-link`** — returns a one-time download URL for a photo. Use when the user explicitly needs to save or download the file. Set `preview = false` (default) for the full-quality original; `preview = true` for a compressed thumbnail. This substitutes for `p6a download` only when the AI client is allowed to fetch and save files from the returned URL.
- **`photo-favorite`** — toggle favorite on a photo.
- **`photo-approve`** — approve a photo in a moderated event (requires moderator permission).

### Sharing

- **`event-browse`** — fetch event details with an inline QR code. Returns the `qrCode` field in the requested `format` (`svg` or `utf8`), plus `url`, favorite state, and `previous`/`next` event IDs for navigation. Also ships as an MCP App widget when the client supports it. Use this when the user wants event details, an inline QR preview, or event navigation. For SVG, save the content to a file before presenting; for UTF-8, render it directly in a code block.
- **`event-qr-link`** — returns a one-time download URL for a full-quality QR code file (PNG, SVG, or UTF-8). Use when the user needs to download or save the QR file. This substitutes for `p6a qr` only when the AI client is allowed to fetch and save files from the returned URL. For in-context QR preview, use `event-browse` instead.
- **`card-gallery`** — fetch a share card preview as an inline base64-encoded JPEG. These are **pre-built themed cards** containing the event's QR code, intended to prompt guests to upload photos — not general-purpose design assets. Also ships as an MCP App widget when the client supports it. Pass `design` (required) and optionally `locale`, `layout`, `paper`, `background`. Use when the user wants to preview the card inline.
  - `bday` — birthday parties and anniversaries
  - `tech` — corporate events, conferences, hackathons
  - `match` — sport events and tournaments
  - `forest` — outdoor and nature-themed events
  - `gold` — luxury and elegant occasions
  - `romantic` — romantic events, weddings, proposals
  - `neon` — parties and nightlife events
- **`card-link`** — returns a one-time download URL for a full-quality card file (PDF or JPG). Use when the user needs to download or print the card. Requires `design`. This substitutes for `p6a card` only when the AI client is allowed to fetch and save files from the returned URL. For in-context card preview, use `card-gallery` instead.

> **MCP Apps note.** `event-browse`, `photo-gallery`, and `card-gallery` ship as MCP App widgets and embed data inline — prefer them for browsing, navigating, and previewing. Use `event-qr-link`, `photo-link`, and `card-link` when the user explicitly needs a download URL for the file, or when the AI client can download from one-time URLs.

---

## Limitations & When to Escape to CLI

MCP is the cleaner surface for in-session work, but it cannot do everything. Switch to `p6a` when:

- **Full-quality file download.** `photo-gallery` and `card-gallery` return compressed previews. `photo-link`, `card-link`, and `event-qr-link` give one-time download URLs — useful to hand off to the user, and equivalent to CLI file operations only when the AI client can fetch and save from those URLs. Otherwise, use `p6a download`, `p6a card`, or `p6a qr`.
- **Bulk sync.** There is no MCP equivalent for `p6a sync` (organized subdirectory layout, idempotent skip-existing logic).
- **Invite link.** No single-purpose tool. Use `event-query` or `event-browse` (both return `url`) as a workaround, or use `p6a link -e <event-id>`.
- **Login/logout.** Credentials are managed outside the MCP session. The MCP server is already authenticated for the active session — there is nothing to "log in" to via tools.
- **Producing a script for the user.** Scripts run outside any agent session, so they must use `p6a`. Even when MCP is available right now, switch to CLI commands when generating output the user will run later.

---

## Common Workflows

### Workflow: Find an Event and Inspect Its Media

```
1. event-query (query = "Birthday")
   → take the first matching event id
2. photo-list (id = <id>)
   → enumerate media items
3. photo-gallery (id = <id>, photo = <photo-id>)
   → fetch inline preview; step through previous/next to navigate
```

When the user says "show me photos from the Birthday event," resolve the event with `event-query` (or `search` for free-form input), then list and preview as needed. Don't dump every preview at once — fetch on demand to keep the context manageable.

### Workflow: Approve All Pending Media in a Moderated Event

```
1. photo-list (id = <id>)
   → filter for items where isApproved == false
2. photo-approve for each pending photo id
```

This is the MCP equivalent of the CLI `jq | xargs` approval loop. Iterate the call rather than batching — there is no bulk approve tool.

### Workflow: Curate Favorites Across an Event

```
1. photo-list (id = <id>)
2. For each candidate the user wants flagged:
   photo-gallery (optional — when viewing or navigating photos helps)
   photo-favorite (id = <id>, photo = <photo-id>, favorite = true)
```

### Workflow: Create and Share a New Event

```
1. event-create (name = "Summer BBQ")
   → capture the returned event id
2. event-browse (id = <id>, format = "svg" or "utf8")
   → shows event details with inline QR; svg: save to file; utf8: render directly
   OR event-qr-link (id = <id>, format = "png")
   → returns a one-time download URL; download it only if the AI client allows that, otherwise hand it to the user
```

If the user wants the QR saved to disk and the AI client cannot download from the one-time URL, switch to CLI: `p6a qr -e <event-id> -t invite.png`.

### Workflow: Hand Off to CLI for Download

```
1. event-query / search → resolve the event id in-session
2. Tell the user: run `p6a download -e <event-id> -t <target-dir>`
```

This is the canonical pattern when the user asks "download these photos" while MCP is connected — use MCP for discovery, then produce a CLI command for the actual file transfer.

---

## Tips

- Treat MCP responses as structured data; don't reformat them into CLI-style tables unless the user asks.
- When an MCP tool is missing for a task, use `fetch` only for simplified event details by ID; otherwise fall back to CLI if the CLI can cover the workflow.
- Event IDs are UUIDs and are interchangeable between MCP and CLI — an id obtained from `event-query` works in `p6a download -e <id>` without translation.
- If the user will rerun the work later (e.g. nightly), don't perform it via MCP — produce a `p6a` script instead so it survives outside the session (if `p6a` is available).
- `event-query` and `event-browse` return `url` — no separate tool call needed to get the invite URL.
