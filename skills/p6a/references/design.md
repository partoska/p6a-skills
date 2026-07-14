# Partoska Design Reference

Use this reference when the user asks you to design an asset tied to a Partoska event — typically a **QR stand**, **poster**, **ticket**, **invitation**, **static HTML page**, **slideshow slide** for a projector/TV, or a photo-driven piece like a **collage** or **photo recap**. See `../SKILL.md` for the full command reference and workflows, and `install.md` for setup.

The job is two-sided: (1) understand the event well enough to design *for* it, not generically; (2) follow the conventions for the specific asset type.

---

## Pre-Built Option: Built-In Share Card Designs

Partoska ships **built-in share card designs** — printable cards with an event QR code that invite guests to upload their photos. They are instant and require no design work.

> **Important**: these cards serve one specific purpose — prompting guests to scan a QR code and upload photos to the event. They are **not** general-purpose design assets. For posters, tickets, invitations, slides, HTML pages, or anything beyond a photo-upload card, always produce a fully custom design (Steps 1–4 below).

Available via `p6a card -e <event-id> -d <design>`:

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

Useful flags:

- `-F jpg` for JPEG output (default: PDF)
- Optional: `-l en|cs`, `-L single|business`, `-p a4|letter`, `-b/--no-background`
- Downloading a share card requires organizer (admin) access.

**When the user asks for a photo-upload card or QR card**, recommend the matching built-in design before offering a custom one. For any other asset type, skip this section entirely and go straight to Step 1.

---

## Step 1: Understand the Event

Before designing anything, gather the minimum context:

- **Name** — the strongest single signal of the event's character.
- **Date(s)** — anchors the season, day-of-week feel, and whether dates should appear on the asset.
- **Optional info / description** — venue, audience, theme, dress code.

If the event already exists in Partoska, fetch it (`p6a list -q "<name>"`) rather than asking the user to retype what's already on file.

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

## Step 2: Consider Using Event Photos

For *any* asset type, ask yourself whether a real photo from the event would make the design stronger.

**Posters benefit most** from real photos — a single hero image carries an enormous amount of vibe with no design effort. Tickets and invitations can use photos but it's optional. QR stands usually do *not* need a photo — they lean minimal and the QR itself is the focal point.

### When the event already has photos

1. List photos with `p6a media -e <event-id>` (add `-F json` to inspect favorite state via jq).
2. Prefer the **most-favorited** ones — they're already validated as visually strong.
3. Download a few candidates in full quality: `p6a download -e <event-id> -m <media-id> -t <path>`.
4. Offer 1–3 thematically appropriate options to the user; let them pick.

Full-quality embedding is straightforward here — `p6a download` fetches the original file, ready to incorporate into the final deliverable.

### When the event is new, empty, or has no relevant photos

Skip this step entirely. Don't ask the user to upload photos just to seed your design — design something thematic on your own.

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
- Get the QR via `p6a qr -e <event-id>` (PNG by default; `-F svg` for a scalable vector to embed in SVG/HTML deliverables).

### Poster

Larger-format announcement (A3/A2 typical) for display before or during the event.

- **Event name dominates** — it's the single largest element.
- **Ask** whether to include start/end date (and time). Posters before the event almost always want dates; posters at the venue may not.
- **QR is secondary** — present so guests can scan, but small relative to the name and imagery. Unless the user explicitly asks for a QR-prominent poster.
- **Photos are the highest-leverage addition here** — see Step 2. Offer if available.
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
- **QR is prominent** when the page is meant for live scanning by guests in the room — large enough to scan from the back row. `p6a qr -e <event-id> -F svg` gives a vector QR to embed inline.
- **Photos work very well here** — see Step 2. A rotating gallery or hero photo can fill the screen.
- Keep CSS self-contained and avoid external font/image dependencies unless the user is hosting it somewhere with internet access.

### Slideshow Slide

A single slide for insertion into an existing PPT/Keynote/Google Slides deck — typically shown between agenda items, at the start/end of a session, or on a loop on a lobby screen.

- **Default to 16:9** unless the user specifies otherwise.
- **Single clear message**: usually event name + QR + a short call to action. Don't crowd it.
- **QR prominent enough to scan** from typical audience distance.
- **Match the host deck's style** if the user can share it — ask if they have an existing template, brand colors, or font to align with. If not, follow the inferred vibe.
- Photos optional; one strong hero image works better than a collage at slide scale.

### Collage / Photo Recap

A photo-driven piece built after (or during) the event — a printed collage, a "best of" recap sheet, or a thank-you page around real photos. This asset type plays to the CLI's strength: `p6a download` fetches full-quality originals directly.

- **Photos are the asset** — start with Step 2: list with `p6a media`, favor the most-favorited, download candidates in full quality.
- **Curate hard**: 3–9 strong photos with room to breathe beat a tiled dump of everything. Let the user confirm the selection.
- **Event name small and secondary** — the photos carry it. Date optional as a caption.
- **QR optional** — a small "relive the event" corner tag works when the recap is shared before the event's upload window closes.
- **Ask about the destination**: print (PDF) vs. screen/social (PNG/JPG or HTML) changes resolution, aspect ratio, and margins.

---

## Step 4: Confirm and Deliver

- When in doubt about tone (formal vs. playful, minimal vs. ornate), confirm with the user before finalizing.
- **Ask about the output format** before producing the final deliverable. Common options:
  - **PDF** — print-ready, vector-preserving, the default for posters / tickets / invitations / QR stands.
  - **PNG / JPG** — raster, good for sharing digitally or embedding in messages.
  - **SVG** — scalable, editable; useful when the user wants to tweak the design themselves.
  - **HTML** — for the static-page asset type, or when the user wants something they can host / open in a browser.
  - **PPTX / Word (DOCX)** — when the user wants to drop the asset into an existing deck or document.
- If you produced multiple variants, label them clearly so the user can pick.

---

## Language Notes

Match the language on the asset to the event's locale — the event name and any user-provided info are the strongest signal. If unsure about the language, ask the user. In any language, keep calls to action conversational and warm, using the natural everyday register rather than overwrought or sentimental wording; reserve formal address and stiff phrasing for corporate, conference, and formal occasions.
