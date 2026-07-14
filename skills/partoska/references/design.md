# Partoska Design Reference

Use this reference when the user asks you to design an asset tied to a Partoska event — typically a **QR stand**, **poster**, **ticket**, **invitation**, **static HTML page**, or **slideshow slide** for a projector/TV. Photo-driven assets — **photo posters, collages, photo recaps/summaries** — are better served by the p6a CLI; see "What You Can Deliver from a Chat Session" below.

The job is two-sided: (1) understand the event well enough to design *for* it, not generically; (2) follow the conventions for the specific asset type.

---

## What You Can Deliver from a Chat Session

Design work over MCP is real, but sharply tiered by whether the asset needs real photos:

- **QR-only assets are the in-session sweet spot.** QR stands, static HTML pages, slideshow slides, invitations, and tickets need typography, color, and the event QR — nothing else. The QR arrives as inline SVG via `event-browse`, so you can author polished SVG and self-contained HTML deliverables entirely in-session. Raster and document output (PNG, PDF, PPTX) additionally depends on the client — use whatever the session provides (artifacts/canvas, image generation, file-creation or code-execution tools, other connected MCP servers); when nothing suitable exists, deliver SVG or HTML and tell the user how to convert (e.g. print-to-PDF from a browser).
- **Built-in share cards shine as MCP Apps.** When the client supports MCP App widgets, present the pre-built designs through the interactive `card-gallery` widget so the user can browse themes visually before committing (see the next section).
- **Photo-driven assets belong to the p6a CLI.** Photo posters, collages, and photo recaps/summaries are built from full-quality originals, and inline photo data will not fit the context in major clients — Claude and ChatGPT both struggle with large base64 payloads. Don't attempt them in-session: design the photo-free variant when one makes sense, and otherwise route the user to the p6a workflow (see Step 2).

---

## Pre-Built Option: Built-In Share Card Designs

Partoska ships **built-in share card designs** — printable cards with an event QR code that invite guests to upload their photos. They are instant and require no design work.

> **Important**: these cards serve one specific purpose — prompting guests to scan a QR code and upload photos to the event. They are **not** general-purpose design assets. For posters, tickets, invitations, slides, HTML pages, or anything beyond a photo-upload card, always produce a fully custom design (Steps 1–4 below).

| Design | Best for |
| --- | --- |
| `bday` | Birthday parties, anniversary celebrations |
| `tech` | Corporate events, conferences, hackathons, tech meetups |
| `match` | Sport events, tournaments, competitions |
| `forest` | Outdoor events, nature-themed gatherings, school trips |
| `garden` | Garden parties, spring celebrations, floral events |
| `gold` | Luxury and elegant occasions |
| `romantic` | Romantic events, weddings, proposals |
| `silver` | Elegant occasions with a cool, modern look |
| `neon` | Parties and nightlife events |
| `nineties` | Retro parties and 1990s-themed events |

Two tools, depending on the use case:

- **`card-gallery`** — opens the interactive card gallery. **Call it once with a starting design and omit `includeData` (or leave it `false`)** — the user browses themes inside the MCP App while the widget fetches preview binaries outside model context. Do not loop over designs with repeated tool calls.
- **`card-link`** — returns a one-time download URL for the full-quality PDF or JPEG. Use when the user needs to download or print the file; if the AI client cannot fetch and save from the URL, hand it to the user to open.

Both tools require event admin access and `design`. Optional values are `locale` (`en` or `cs`), `layout` (`single` or `business`), `paper` (`a4` or `letter`), and boolean `background`; `card-link` additionally accepts `format` (`pdf` or `jpg`).

**When the user asks for a photo-upload card or QR card**, recommend the matching built-in design before offering a custom one. For any other asset type, skip this section entirely and go straight to Step 1.

---

## Step 1: Understand the Event

Before designing anything, gather the minimum context:

- **Name** — the strongest single signal of the event's character.
- **Date(s)** — anchors the season, day-of-week feel, and whether dates should appear on the asset.
- **Optional info / description** — venue, audience, theme, dress code.

If the event already exists in Partoska, fetch it (`event-query`) rather than asking the user to retype what's already on file.

### Inferring the Vibe

From name + date + any provided info, infer what kind of event this is. Common categories:

- **Wedding** — names like "Anna & Petr", "Wedding 2026", weekend dates in spring/summer.
- **Family birthday / anniversary** — "Marek's 40th", "Grandma 80", "10 Years Together".
- **Kids' party** — "Ella's 5th", themed names ("Pirate Party").
- **Sport event** — "Spartan Race Brno", "Company Football Cup", race / tournament wording.
- **Company outing / team event** — "Q4 Offsite", "Team Building 2026".
- **Conference / corporate** — "DevSummit 2026", "Annual Kickoff".
- **Concert / festival** — venue or band name, evening dates.
- **Graduation, baptism, school event** — institution name, age cues.

The vibe drives typography, palette, and tone:

- Wedding → elegant serif, soft palette, romantic motifs (florals, monograms).
- Kids' party → playful sans, bright colors, illustrative motifs.
- Corporate → restrained sans, brand-safe palette, minimal ornament.
- Sport → bold/condensed type, high-contrast palette, dynamic motifs.
- Family celebration → warm/personal, photo-friendly layouts.

**If you're not confident in the vibe, ask.** A short clarifying question ("Is this a wedding reception or an engagement party?", "Is the audience adults or kids?") is far cheaper than producing an asset in the wrong register. Don't blindly guess between two plausible categories.

---

## Step 2: Photos — Default to None In-Session

For the assets you design in-session (QR stand, invitation, ticket, slide, HTML page, photo-free poster), the default is a **photo-free design** — typography, color, and motif carry the vibe. Don't pull photo data into context unless it genuinely changes the outcome.

When a photo would help:

### Previews to inform, sparingly

1. List photos with `photo-list`. Prefer the **most-favorited** ones — they're already validated as visually strong.
2. Open `photo-gallery` once with a starting item, omitting `includeData` (or leaving it `false`). Let the user browse candidates inside the MCP App; do not fetch each preview through separate tool calls.
3. Continue after the user selects a candidate in the widget, using their selection and the event metadata to guide palette, mood, and composition.

### Embedding full quality: hand off to p6a

`photo-gallery` previews are not embedding quality, and the one-time URLs from `photo-link` expire after a single use — never hard-code one into a deliverable. If the user wants real photos *in* the deliverable — a photo poster, collage, or recap — that's a p6a task: point them to `p6a download -e <event-id> -t <dir>` (or `-m <media-id>` for one file). If it helps, still deliver the design in-session as a template with clearly marked photo placeholders they can fill.

### When the event is new, empty, or has no relevant photos

Skip photos entirely. Don't ask the user to upload photos just to seed your design — design something thematic on your own.

---

## Step 3: Design the Asset

Conventions per asset type. In all cases, match typography and color to the inferred vibe — no generic AI gradients or stock-template aesthetics.

### QR Stand

A small printed sign (typically table-card or A5/A6) inviting guests to upload their photos via the event's QR code.

- **Aesthetic**: minimalistic. The QR code is the focal point.
- **Do not** mention partoska.com or promote the service. The QR is the product; users don't need branding pushed at them.
- **Call to action**: warm, personal, in the host's voice. Example:
  - *"Help us remember these moments — share your photos with us."*
  - Match language to the event's locale.
- **Photo contest hook** (e.g. "Best photo wins X"): **ask the user first** before adding this. Never invent prizes unprompted — the host needs to actually deliver on whatever you put on the sign.
- **Photos**: usually unnecessary on a QR stand. Offer only if the user asks.
- Get the QR via `event-browse` — the inline `svg` format embeds directly into an SVG or HTML deliverable (also an MCP App widget); `utf8` renders in a code block. Use `event-qr-link` when the user needs a standalone QR file (one-time download URL for PNG/SVG/UTF-8).

### Poster

Larger-format announcement (A3/A2 typical) for display before or during the event.

- **Event name dominates** — it's the single largest element.
- **Ask** whether to include start/end date (and time). Posters before the event almost always want dates; posters at the venue may not.
- **QR is secondary** — present so guests can scan, but small relative to the name and imagery. Unless the user explicitly asks for a QR-prominent poster.
- **A typography-led thematic poster is achievable in-session.** Real photos are the highest-leverage addition, but they turn the poster into a p6a task — see Step 2.
- Style follows the inferred vibe.

### Ticket

A handout or printable per-guest pass.

- **Event name prominent**, but balanced against practical info (date, time, seat/table if relevant).
- **Ask** about including dates and times — usually yes for tickets, but confirm.
- **QR is small / secondary** by default. If the QR is for entry validation rather than photo upload, that's a different feature — clarify with the user.
- Photos optional; a small motif or band along one edge often works better than a full hero image.

### Invitation

Sent to guests ahead of the event (printed card or digital).

- **Event name + date** are both expected — invitations without a date are useless.
- **More room for thematic flourish**: illustration, monogram, motif tied to the vibe (florals for weddings, balloons for kids' parties, etc.).
- **QR small / secondary** by default — guests will return to the invitation later to find logistics, not to scan.
- Tone follows the vibe heavily: weddings formal/elegant, family parties warm/casual.
- Photos optional; engagement photos for weddings are a common ask — offer if available.

### Static HTML Page

A self-contained page intended to be displayed on a screen (projector, TV, lobby monitor) or shared via a link. Single `.html` file with inline CSS is usually best — easy for the user to host or open locally.

- **Aspect ratio matters** — ask whether the target is 16:9 (most TVs/projectors), 9:16 (vertical signage), or browser-flexible. Design accordingly.
- **Event name dominates**, similar to a poster.
- **QR is prominent** when the page is meant for live scanning by guests in the room — large enough to scan from the back row. Embed the SVG QR from `event-browse` inline; never reference a one-time URL from the page.
- **Photos work very well here** — but a photo-filled page (rotating gallery, hero photo) is a p6a task; see Step 2. In-session, deliver the page with marked photo placeholders or design it QR-and-typography-first.
- Keep CSS self-contained and avoid external font/image dependencies unless the user is hosting it somewhere with internet access.

### Slideshow Slide

A single slide for insertion into an existing PPT/Keynote/Google Slides deck — typically shown between agenda items, at the start/end of a session, or on a loop on a lobby screen.

- **Default to 16:9** unless the user specifies otherwise.
- **Single clear message**: usually event name + QR + a short call to action. Don't crowd it.
- **QR prominent enough to scan** from typical audience distance.
- **Match the host deck's style** if the user can share it — ask if they have an existing template, brand colors, or font to align with. If not, follow the inferred vibe.
- Photos optional; one strong hero image works better than a collage at slide scale.

---

## Step 4: Confirm and Deliver

- When in doubt about tone (formal vs. playful, minimal vs. ornate), confirm with the user before finalizing.
- **Ask about the output format** before producing the final deliverable, and offer only formats the session can actually produce (see "What You Can Deliver from a Chat Session" above). Common options:
  - **SVG** — scalable, editable, always producible in chat; useful when the user wants to tweak the design themselves.
  - **HTML** — for the static-page asset type, or when the user wants something they can host / open in a browser; also always producible.
  - **PDF** — print-ready, vector-preserving, the natural default for posters / tickets / invitations / QR stands — when the client has a file-producing capability. Otherwise deliver SVG/HTML and suggest print-to-PDF from a browser.
  - **PNG / JPG** — raster, good for sharing digitally; requires a rendering capability in the client (image generation, code execution).
  - **PPTX / Word (DOCX)** — when the user wants to drop the asset into an existing deck or document; requires a file-producing capability.
- If you produced multiple variants, label them clearly so the user can pick.

---

## Language Notes

Match the language on the asset to the event's locale — the event name and any user-provided info are the strongest signal. If unsure about the language, ask the user. In any language, keep calls to action conversational and warm, using the natural everyday register rather than overwrought or sentimental wording; reserve formal address and stiff phrasing for corporate, conference, and formal occasions.
