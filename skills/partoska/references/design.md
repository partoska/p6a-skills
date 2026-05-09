# Partoska Design Reference

Use this reference when the user asks you to design an asset tied to a Partoska event — typically a **QR stand**, **poster**, **ticket**, **invitation**, **static HTML page**, or **slideshow slide** for a projector/TV. See `../SKILL.md` for the broader CLI-vs-MCP routing.

The job is two-sided: (1) understand the event well enough to design *for* it, not generically; (2) follow the conventions for the specific asset type.

---

## Step 1: Understand the Event

Before designing anything, gather the minimum context:

- **Name** — the strongest single signal of the event's character.
- **Date(s)** — anchors the season, day-of-week feel, and whether dates should appear on the asset.
- **Optional info / description** — venue, audience, theme, dress code.

If the event already exists in Partoska, fetch it (`event-query` / `p6a list -q`) rather than asking the user to retype what's already on file.

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

### Theme Colors and Branding

For certain event types, there is usually an intentional color story or identity — ask before designing:

- **Weddings**: couples commonly choose a theme color (gold, champagne, beige, dusty rose, violet, sage green, white, etc.). Ask: *"Do you have a wedding color palette or theme?"* If they don't, default to soft and neutral.
- **Sports teams**: teams have colors and often a logo. Ask for both. If they can't provide a logo file, at minimum get the primary and secondary colors so the design feels like theirs rather than generic.
- **Companies / corporate events**: brands have color guidelines and logos. Ask for a logo file or brand colors. If neither is available, keep the palette neutral and professional rather than inventing colors that might clash with the actual brand.

Don't ask about theme colors for kids' parties, family celebrations, or other casual events where no brand identity is expected — infer from the vibe or ask only if the design feels incomplete.

---

## Step 2: Consider Using Event Photos

For *any* asset type, ask yourself whether a real photo from the event would make the design stronger.

**Posters benefit most** from real photos — a single hero image carries an enormous amount of vibe with no design effort. Tickets and invitations can use photos but it's optional. QR stands usually do *not* need a photo — they lean minimal and the QR itself is the focal point.

### When the event already has photos

1. List photos with `event-photo-list` (MCP) or `p6a media -e <event>` (CLI).
2. Prefer the **most-favorited** ones — they're already validated as visually strong.
3. Inspect candidates with `event-photo-preview` (MCP) or by downloading a few with `p6a download` (CLI).
4. Offer 1–3 thematically appropriate options to the user; let them pick.

### When the event is new, empty, or has no relevant photos

Skip this step entirely. Don't ask the user to upload photos just to seed your design — design something thematic on your own.

### MCP caveat — full-quality photos are CLI-only

The MCP `event-photo-preview` tool returns a **low-resolution preview only**, suitable for *picking* a thematic candidate but not for embedding in a final deliverable. In MCP-only sessions, default to **skipping the photo step**. If the user really wants a specific photo in the design, point them to `p6a download -e <event-id> -m <media-id> -t <path>` to fetch the full file themselves, then incorporate it.

In CLI-driven sessions where you (or the user) can run `p6a download`, full-quality embedding is straightforward.

---

## Step 3: Design the Asset

Conventions per asset type. In all cases, match typography and color to the inferred vibe — no generic AI gradients or stock-template aesthetics.

### Print vs. Digital Background Rule

**Printed materials** (QR stands, invitations, tickets, and printed posters): keep the background **dominantly white**. Most users will print on standard white paper, or on decorative paper they chose specifically for its own texture and color. A dark or heavily colored background wastes ink, fights the paper, and often looks worse than a clean white design. Decorative elements, color accents, and typography can carry the theme without flooding the background. The exception is if the user explicitly says they are printing professionally on pre-colored stock and wants a full-bleed design.

**Digital materials** (static HTML pages, slideshow slides, and anything displayed on a screen): this constraint does not apply. Rich backgrounds, dark modes, and full-bleed color all work fine.

### QR Stand

A small printed sign (typically table-card or A5/A6) inviting guests to upload their photos via the event's QR code.

- **Aesthetic**: minimalistic. The QR code is the focal point.
- **Do not** mention partoska.com or promote the service. The QR is the product; users don't need branding pushed at them.
- **Call to action**: warm, personal, in the host's voice. Example:
  - *"Help us remember these moments — share your photos with us."*
  - Match language to the event's locale.
- **Photo contest hook** (e.g. "Best photo wins X"): **ask the user first** before adding this. Never invent prizes unprompted — the host needs to actually deliver on whatever you put on the sign.
- **Photos**: usually unnecessary on a QR stand. Offer only if the user asks.
- Get the QR PNG via `event-share-qr` (MCP) or `p6a qr -e <event-id>` (CLI).

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
- **QR is prominent** when the page is meant for live scanning by guests in the room — large enough to scan from the back row.
- **Photos work very well here** — see Step 2. A rotating gallery or hero photo can fill the screen.
- Keep CSS self-contained and avoid external font/image dependencies unless the user is hosting it somewhere with internet access.

### Slideshow Slide

A single slide for insertion into an existing PPT/Keynote/Google Slides deck — typically shown between agenda items, at the start/end of a session, or on a loop on a lobby screen.

- **Default to 16:9** unless the user specifies otherwise.
- **Single clear message**: usually event name + QR + a short call to action. Don't crowd it.
- **QR prominent enough to scan** from typical audience distance.
- **Match the host deck's style** if the user can share it — ask if they have an existing template, brand colors, or font to align with. If not, follow the inferred vibe.
- Photos optional; one strong hero image works better than a collage at slide scale.

---

## Step 4: Preview and Self-Critique

If you have the capability to render or preview the output (open a PDF, view an image, render HTML in a browser), do it — then critique the result before handing it to the user. Look for:

- **Layout**: alignment, whitespace, visual hierarchy. Does the most important element (event name, QR) actually read first? Are margins consistent? Does anything feel cramped or floating?
- **Clarity**: can a guest immediately understand what this is and what to do? Is the call to action obvious?
- **Readability**: sufficient contrast between text and background, font size appropriate for the intended viewing distance, no text overlapping imagery.
- **Anti-AI-sloppiness**: over-decorated gradients, mismatched font styles, Lorem Ipsum fragments left in, stock-template flourishes that have nothing to do with the event, filler copy like "Your event name here", hallucinated details not in the event data, or generic placeholder imagery. If you spot any of these, fix before delivering.

If you cannot render or preview the output, skip this step and note the limitation to the user.

---

## Step 5: Confirm and Deliver

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

Match the language on the asset to the event's locale (event name and any user-provided info are the strongest signal). Below are language-specific conventions worth following. If unsure about language, ask user.

### Czech

When designing in Czech:

- **Use "fotky" for photos**, not "vzpomínky" ("memories"). "Vzpomínky" reads as overwrought / sentimental in this context — "fotky" is the natural everyday word people use.
- Keep calls to action conversational and warm, e.g. *"Pomoz nám zachytit tenhle den — nahraj svoje fotky."* Avoid stiff or formal phrasing unless the event itself is formal.
- Address guests informally (tykání) by default for weddings, family events, parties; use formal address (vykání) for corporate, conferences, and formal occasions.
