---
name: partoska
description: Help users manage Partoska.com photo-sharing events through the Partoska MCP server — find and create events, browse/preview/upload/curate photos, review photos on the user's behalf (moderate uploads, pick favorites, privacy-check before going public, find a moment), help hosts during a live event (coverage checks, guest nudges, live moderation), share via invite links, QR codes, and printable cards, and design event assets including photo collages and recap stories. Use when Partoska MCP tools (e.g. event-query, photo-list) are available in the session.
license: MIT
metadata:
  author: Partoska Laboratory
  version: "3.0.1"
---

# Partoska Skill

## Usage

- `/partoska` — describe what you want to do; the assistant will pick the right tools
- `/partoska show me photos from event B-Day`
- `/partoska create an event for Saturday's BBQ and give me the QR code`
- `/partoska make a QR stand for the wedding`
- `/partoska make a collage from the best photos of the BBQ`
- `/partoska download the favorite photos from Anna's wedding`
- `/partoska moderate the pending photos at the company party — nothing with drinks or logos`
- `/partoska revoke approval of the photo with the receipt at the gala`
- `/partoska favorite the 10 best photos from the BBQ`
- `/partoska make the wedding public — check the photos first`
- `/partoska find the cake cutting photos from Ella's party`
- `/partoska how are the photos going at the party? nudge the guests`
- `/partoska keep moderating the gala every 20 minutes until midnight`
- `/partoska make a recap story of the hike`

## What This Skill Does

This skill makes you an expert on [Partoska.com](https://partoska.com) — a photo-sharing service for events — through the Partoska MCP server (`https://api.partoska.com/mcp/v1`). It covers finding and managing events, browsing and curating photos, uploading, moderation, sharing via invite links, QR codes, and printable cards, and designing event assets. You can also look at photos yourself and act for the user — moderating, curating, privacy-checking — and help a host keep a live event's photo collection going.

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
- **`event-update`** — partially update an event. Send only the fields you intend to change (`name`, `public`, `favorite`, `moderated`, `from`/`to`). All fields except `favorite` require admin access; `favorite` stars the event for the current user and needs only guest access.

### Photo Inspection & Curation

- **`photo-list`** — list photos for a given event. Use to enumerate before previewing or curating. Check `isApproved` to identify pending photos in a moderated event.
- **`photo-gallery`** — open an event's interactive photo gallery with metadata, favorite state and count, and navigation. Call it once with a starting photo, omit `includeData` (or leave it `false`), and let the user browse through the MCP App; the widget fetches preview binaries without loading them into model context. `includeData = true` returns the preview as base64 data: called directly, it only helps clients that render it as an image (many return base64 text you cannot see), but called from a code sandbox that writes it to a file, it is a data-transfer path when the shell can't fetch URLs. To look at photos yourself, follow "Seeing a Photo" in `references/review.md`. For a full-quality download link, use `photo-link`.
- **`photo-link`** — returns a one-time download URL for a photo. Use when the user needs to save or download the file, or when you need the photo itself in your scratch space — for a collage once the user has picked it, or with `preview = true` to look at photos yourself (see `references/review.md`). Set `preview = false` (default) for the full-quality original; `preview = true` for a compressed thumbnail.
- **`photo-picker`** — upload a photo or video to an event; also ships as an MCP App widget. When called with only `id`, returns event context (`event`, `name`, `url`, `guests`, `photos`, `allowVideo`, `quota`, `photo: null`) so the MCP App can present the upload UI. When called with `id` + `type` (`image/jpeg`, `video/mp4`, or `video/quicktime`) + `data` (base64 data URL, e.g. `data:image/jpeg;base64,...`), performs the upload and returns the new photo identifier in the `photo` field. In moderated events the upload is created as unapproved until a moderator approves it. **Prefer the MCP App widget path (call with `id` only and let the App handle the rest) over passing `data` directly whenever the client supports it** — base64-encoded photos and videos are large and will bloat the context window.
- **`photo-favorite`** — mark (`favorite = true`) or unmark (`favorite = false`) a photo as a favorite for the current user. Explicit, not a toggle.
- **`photo-approve`** — approve a photo in a moderated event, making it visible to the event's audience — for a public event, that includes anonymous visitors with the invite link — or with `approved = false` put it back on hold, hiding it again from everyone except its author and moderators. Returns the photo's current `isApproved`. Requires moderator permission; only moderated events accept `approved = false`. There is no tool to reject or delete a photo.

> **Photo access note.** `photo-list`, `photo-gallery`, `photo-picker`, and `photo-link` need guest access on a public event, and contributor access on a non-public one.

> **Quota note.** The `quota` returned by `photo-picker` is the event's *remaining* upload capacity in bytes, not a fixed ceiling. 

### Sharing

- **`event-browse`** — fetch event details with an inline QR code. Returns the `qrCode` field in the requested `format` (`svg` or `utf8`), plus `url`, favorite state, and `previous`/`next` event IDs for navigation. Also ships as an MCP App widget when the client supports it. Use this when the user wants event details, an inline QR preview, or event navigation. For SVG, save the content to a file (or embed it in an SVG/HTML deliverable) before presenting; for UTF-8, render it directly in a code block.
- **`event-qr-link`** — returns a one-time download URL for a full-quality QR code file (PNG, SVG, or UTF-8). Use when the user needs to download or save the QR file. For in-context QR preview, use `event-browse` instead.
- **`card-gallery`** — open the interactive share-card gallery. These are **pre-built themed cards** containing the event's QR code, intended to prompt guests to upload photos — not general-purpose design assets. Call it once with a starting design, omit `includeData` (or leave it `false`), and let the user browse designs through the MCP App; the widget fetches preview binaries without loading them into model context. Requires event admin access and `design`; optional values are `locale` (`en`, `cs`, `sk`, `pl`, `ru`, `es`, `de`, `fr`, `it`, `nl`, or `uk`), `layout` (`single` or `business`), `paper` (`a4` or `letter`), and boolean `background`.
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
  - `cottage` — cottage getaways, cabin weekends, countryside retreats
  - `wish` — birthday parties and milestone celebrations with balloons and confetti
  - `velvet` — formal balls, charity galas, ballroom evenings
  - `hallows` — Halloween costume parties, pumpkin-carving nights, spooky gatherings
- **`card-link`** — returns a one-time download URL for a full-quality card file. Requires event admin access and `design`; accepts `format` (`pdf` or `jpg`) plus the same locale, layout, paper, and background options as `card-gallery`. Use when the user needs to download or print the card. For an in-context preview, use `card-gallery` instead.

> **MCP Apps note.** `event-browse`, `photo-gallery`, `photo-picker`, and `card-gallery` ship as MCP App widgets — prefer them for browsing, navigating, previewing, and uploading. Open a gallery once, keep `includeData` omitted or `false`, and let the user browse inside the App; do not loop over gallery items with repeated tool calls. To look at photos yourself, follow "Seeing a Photo" in `references/review.md`. Use `event-qr-link`, `photo-link`, and `card-link` when the user needs the file itself, or you need it in your scratch space to build a deliverable.

> **One-time URL note.** The URLs returned by `photo-link`, `card-link`, and `event-qr-link` are single-use and expire. They are the primary path to full-quality files: when the AI client can fetch URLs (code execution, a sandbox, or file tools), download each one right away into its scratch space and work from the local file — never pull the bytes into the conversation as base64. When it can't, hand the URL to the user to open themselves. Never embed a one-time URL into a deliverable (e.g. an HTML page) that will be opened more than once.

---

## Common Workflows

### Workflow: Find an Event and Inspect Its Media

```
1. event-query (query = "Birthday")
   → take the first matching event id
2. photo-list (id = <id>)
   → enumerate media items
3. photo-gallery (id = <id>, photo = <photo-id>)
   → open the MCP App once; the user browses and selects photos inside the gallery
```

When the user says "show me photos from the Birthday event," resolve the event with `event-query` (or `search` for free-form input), list the media, and open `photo-gallery` with a starting item. Let the user browse in the widget; don't issue one tool call per photo.

### Workflow: Approve All Pending Media in a Moderated Event

```
1. photo-list (id = <id>)
   → filter for items where isApproved == false
2. photo-approve for each pending photo id
```

Iterate the call rather than batching — there is no bulk approve tool. Use this only when the user explicitly wants everything approved unseen; when they ask you to *moderate*, *check*, or *review* the uploads, follow `references/review.md`.

### Workflow: Curate Favorites Across an Event

```
1. photo-list (id = <id>)
   → choose a starting photo id
2. photo-gallery (id = <id>, photo = <photo-id>)
   → open the MCP App once; the user browses and toggles favorites in the gallery
```

The gallery widget invokes `photo-favorite` for the user's toggles. Call `photo-favorite` directly when the App is unavailable, when the user provides a specific photo ID without needing to browse, or when the user asks you to pick favorites yourself (see `references/review.md`).

### Workflow: Work with Full-Quality Photos

```
1. photo-list (id = <id>)
   → pick photos (favorites first) or let the user pick in photo-gallery
2. photo-link (id = <id>, photo = <photo-id>) for each pick
   → fetch each one-time URL immediately into the client's scratch space
3. work from the local files (collage, recap, resize, analysis)
   → to hand the photos themselves to the user, package them (e.g. a zip) as a downloadable file
```

Never pull photo bytes into the conversation as base64. If the client can't fetch URLs, hand the links to the user instead.

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

## Task Guides

Load the matching reference **before** starting any of these tasks:

| Task | Reference |
| --- | --- |
| Looking at photos yourself and acting on them — moderating pending uploads, picking favorite photos or events, a privacy check before making an event public, finding a specific moment. **Requires** either a shell or code execution that can download files, or a code sandbox that can call MCP tools and write files; plus an image-viewing or vision tool | `references/review.md` |
| Helping a host during a live event — how photo collection is going, gaps in coverage, nudging guests to upload, live moderation on a schedule | `references/hosting.md` |
| Designing an asset — QR stand, poster, ticket, invitation, HTML page, slide, collage, photo recap, recap story — or recommending a built-in share card | `references/design.md` |

Listing photos, event settings, QR codes, cards, and sharing need no vision. Neither do explicit user commands to approve, put on hold, favorite, or unfavorite identified photos, or to approve all pending media without review. Resolve the exact event and items and follow the user's requested scope. Anything that judges photo content — moderating against rules, choosing favorites, the privacy check, finding a moment, live moderation — requires you to actually see the photos. If the session can't (see `references/review.md`), explain that limit and do not make content-based decisions or claims about unseen photos.

When the user asks to make an event public, offer the privacy check from `references/review.md` before calling `event-update (public = true)`.

---

## Tips

- Treat MCP responses as structured data defined by each tool's output schema; don't assume field names.
- Need an invite URL? `event-query` and `event-browse` both return `url` — no separate tool call needed.
- Use `fetch` only for simplified event details by ID (ChatGPT-compatible result shape).
