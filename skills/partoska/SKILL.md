---
name: partoska
description: Help users manage Partoska.com photo-sharing events through the Partoska MCP server — find and create events, browse/preview/upload/curate photos, approve moderated uploads, share via invite links, QR codes, and printable cards, and design event assets. Use when Partoska MCP tools (e.g. event-query, photo-list) are available in the session.
license: MIT
metadata:
  author: Partoska Laboratory
  version: "2.0.0"
---

# Partoska Skill

## Usage

- `/partoska` — describe what you want to do; the assistant will pick the right tools
- `/partoska show me photos from event B-Day`
- `/partoska create an event for Saturday's BBQ and give me the QR code`
- `/partoska make a QR stand for the wedding`

## What This Skill Does

This skill makes you an expert on [Partoska.com](https://partoska.com) — a photo-sharing service for events — through the Partoska MCP server (`https://api.partoska.com/mcp/v1`). It covers finding and managing events, browsing and curating photos, uploading, moderation, and sharing via invite links, QR codes, and printable cards.

The tools below are available when Partoska MCP tools (e.g. `event-query`, `photo-list`) are visible in the session. If they are not, tell the user to connect their AI assistant to the Partoska MCP server at `https://api.partoska.com/mcp/v1` — exact steps depend on the assistant, typically an "Add MCP server" or "Connectors" setting.

---

## Available Tools

Tool names below are the canonical Partoska MCP tool names. Different MCP clients may surface them under host-specific prefixes (e.g. `partoska/event-query`, `mcp.partoska.event-query`) — match by the trailing tool name. Output schemas are defined by the server — consult each tool's schema for exact field names and types.

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
- **`photo-link`** — returns a one-time download URL for a photo. Use when the user explicitly needs to save or download the file. Set `preview = false` (default) for the full-quality original; `preview = true` for a compressed thumbnail.
- **`photo-picker`** — upload a photo or video to an event; also ships as an MCP App widget. When called with only `id`, returns event context (`event`, `name`, `url`, `guests`, `photos`, `allowVideo`, `quota`, `photo: null`) so the MCP App can present the upload UI. When called with `id` + `type` (`image/jpeg`, `video/mp4`, or `video/quicktime`) + `data` (base64 data URL, e.g. `data:image/jpeg;base64,...`), performs the upload and returns the new photo identifier in the `photo` field. In moderated events the upload is created as unapproved until a moderator approves it. **Prefer the MCP App widget path (call with `id` only and let the App handle the rest) over passing `data` directly whenever the client supports it** — base64-encoded photos and videos are large and will bloat the context window.
- **`photo-favorite`** — toggle favorite on a photo.
- **`photo-approve`** — approve a photo in a moderated event (requires moderator permission).

### Sharing

- **`event-browse`** — fetch event details with an inline QR code. Returns the `qrCode` field in the requested `format` (`svg` or `utf8`), plus `url`, favorite state, and `previous`/`next` event IDs for navigation. Also ships as an MCP App widget when the client supports it. Use this when the user wants event details, an inline QR preview, or event navigation. For SVG, save the content to a file (or embed it in an SVG/HTML deliverable) before presenting; for UTF-8, render it directly in a code block.
- **`event-qr-link`** — returns a one-time download URL for a full-quality QR code file (PNG, SVG, or UTF-8). Use when the user needs to download or save the QR file. For in-context QR preview, use `event-browse` instead.
- **`card-gallery`** — fetch a share card preview as an inline base64-encoded JPEG. These are **pre-built themed cards** containing the event's QR code, intended to prompt guests to upload photos — not general-purpose design assets. Also ships as an MCP App widget when the client supports it. Pass `design` (required) and optionally `locale`, `layout`, `paper`, `background`. Use when the user wants to preview the card inline.
  - `bday` — birthday parties and anniversaries
  - `tech` — corporate events, conferences, hackathons
  - `match` — sport events and tournaments
  - `forest` — outdoor and nature-themed events
  - `garden` — garden parties, spring celebrations, floral events
  - `gold` — luxury and elegant occasions
  - `romantic` — romantic events, weddings, proposals
  - `silver` — elegant occasions with a cool, modern look
  - `neon` — parties and nightlife events
  - `nineties` — retro parties and 1990s-themed events
- **`card-link`** — returns a one-time download URL for a full-quality card file (PDF or JPG). Use when the user needs to download or print the card. Requires `design`. For in-context card preview, use `card-gallery` instead.

> **MCP Apps note.** `event-browse`, `photo-gallery`, `photo-picker`, and `card-gallery` ship as MCP App widgets and embed data inline — prefer them for browsing, navigating, previewing, and uploading. Use `event-qr-link`, `photo-link`, and `card-link` when the user explicitly needs a download URL for the file.

> **One-time URL note.** The URLs returned by `photo-link`, `card-link`, and `event-qr-link` are single-use and expire. They are the primary path to full-quality files when the AI client is allowed to fetch and save from URLs; when it isn't, hand the URL to the user to open themselves. Never embed a one-time URL into a deliverable (e.g. an HTML page) that will be opened more than once.

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

Iterate the call rather than batching — there is no bulk approve tool.

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
   → shows event details with inline QR; svg: save to file or embed; utf8: render directly
   OR event-qr-link (id = <id>, format = "png")
   → returns a one-time download URL; download it only if the AI client allows that, otherwise hand it to the user
```

---

## Beyond MCP: the p6a CLI

Partoska also ships **p6a**, a command-line tool available at [github.com/partoska/p6a-cmd](https://github.com/partoska/p6a-cmd) and installable, e.g., via `npm install -g @partoska/p6a`. MCP covers nearly everything in-session; point users to p6a only when:

- **Bulk backup.** There is no MCP equivalent of `p6a sync`, which downloads all events into organized subdirectories and safely skips already-downloaded files.
- **Full-quality files the client can't fetch.** When the AI client cannot fetch and save from the one-time URLs returned by `photo-link` / `card-link` / `event-qr-link`, hand the URL to the user — or for many files, suggest `p6a download -e <event-id> -t <dir>`.
- **Scripts, cron jobs, CI.** Anything the user will run outside an agent session must use `p6a` commands — MCP tools only exist inside an active agent session.
- **Photo-driven design assets.** Photo posters, collages, and photo recaps are built from full-quality originals that won't fit the context inline — resolve the event in-session, then hand off to `p6a download` (see `references/design.md`).

Event IDs are UUIDs and interchangeable between surfaces — an id obtained from `event-query` works in `p6a download -e <id>` without translation.

---

## Design Tasks

Partoska users frequently ask for design assets tied to their events — most commonly **QR stands** (table signage that points guests to the event's photo upload page), **posters**, **tickets**, **invitations**, and screen-displayed assets like **static HTML pages** or **slideshow slides** (projector / TV / lobby display).

When the user requests any of these, **load `references/design.md` first** before producing anything. It covers what is deliverable in-session — QR-only assets (stands, HTML pages, slides, invitations, tickets) are the sweet spot since the QR arrives as inline SVG via `event-browse`, and built-in share cards present best through the interactive `card-gallery` MCP App widget — plus vibe inference from event metadata, asset-specific conventions, and when to route photo-driven assets (photo posters, collages, photo recaps) to the p6a CLI.

---

## Tips

- Treat MCP responses as structured data defined by each tool's output schema; don't assume field names.
- Need an invite URL? `event-query` and `event-browse` both return `url` — no separate tool call needed.
- Use `fetch` only for simplified event details by ID (ChatGPT-compatible result shape).
- If the user will rerun the work later (e.g. nightly), don't perform it via MCP — produce a `p6a` script instead so it survives outside the session.
