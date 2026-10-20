# Helping a Host Around a Live Event

Use this reference when the user is hosting an event that is happening now, or about to: checking how photo collection is going, nudging guests to upload, or keeping moderation running while they enjoy the party. See `../SKILL.md` for the tool reference and `review.md` for how to look at and judge photos.

---

## Workflow: Coverage Check and Guest Nudge

"How are the photos going?", "Did anyone take pictures of the speeches?", "Remind people to upload."

```
1. event-query (query = "<event name>")
   → event window (from/to), guest and photo counts, moderated state, invite url
2. photo-list (id = <id>, sort = "taken-asc")
   → taken/uploaded timestamps and approval state for every photo
3. summarize, then offer a nudge
```

What to report, briefly:

- **Volume.** Photos so far versus guests who joined, and how many arrived in the last hour.
- **Gaps in time.** Bucket photos by taken time (e.g. 30-minute slots) across the event window and point out empty or thin stretches — "nothing between 21:00 and 22:30." Taken timestamps come from the camera and may be in UTC or off by a time zone; say so if the gaps look suspicious.
- **Gaps in moments.** If the user names moments that matter (speeches, first dance, award ceremony), check whether photos exist around those times. To confirm, look at a few photos from around those times — `photo-link (preview = true)` → download → view, as in "Seeing a Photo" in `review.md` — don't scan the whole event.
- **Moderation backlog.** In a moderated event, the number of photos still pending. Other guests can't see those uploads until they are approved, and a gallery that looks empty discourages more uploads — offer to moderate (`review.md`).

The photo list doesn't reliably tell you *which* guests have uploaded, so don't name people who haven't.

Then offer a nudge the host can send or show:

- **A message** for the event's group chat: short, warm, in the event's language, with the invite `url`. If a moment is missing, ask for it specifically — "Anyone got photos from the first dance? Add them here: <url>".
- **A screen slide or table card** with the QR, when guests are in the room — see the slideshow slide and QR stand guidance in `design.md`.

Draft the message; don't send it yourself. If the session has a messaging or email tool and the user asks you to send it, show the final text and the recipients and get a yes first.

---

## Workflow: Live Moderation

Keep a moderated event flowing during the party: review new uploads every 15–30 minutes, so guests see their photos quickly without the host checking a phone all evening. It needs a client that can run a task repeatedly or on a schedule; if the client can't, offer to run a moderation pass whenever the user asks instead.

**Set up — with the user, before the first run:**

1. Resolve the event and check `isModerated` in `event-query`; on an unmoderated event uploads are already visible, so there's nothing to moderate. Confirm the session can see photos ("Before You Start" in `review.md`).
2. Agree on the rules (see "Ground Rules" in `review.md`). Nobody will be around to answer questions mid-run, so settle them now; whatever the rules don't clearly cover stays pending.
3. Agree on the interval, when to stop (default: the event's `to` time), and how to report — by default a short note only when something was flagged, plus a summary at the end.

**Each run:**

```
1. photo-list (id = <id>, sort = "uploaded-asc")
   → pending items (isApproved == false)
2. skip photos already judged in earlier runs, if your notes from those runs are available
3. review and approve the new ones as in "Workflow: Moderate Pending Photos" (review.md)
4. log the run: time, number approved, photos newly flagged with reasons
```

- **Carry state when you can.** If the client keeps notes or files between runs, record each judged photo id and decision so later runs only look at new uploads. If it doesn't, re-judging still-pending photos is harmless — just don't report the same flagged photo again.
- **Stop on trouble.** If approving is refused, or the event stops being moderated, stop the schedule and tell the user.
- **Fixing a slip.** If the user spots a photo that shouldn't have been approved, put it back on hold as in `review.md` — and tighten the rules for the remaining runs.
- **Wrap up.** At the end, summarize the whole run — approved, still pending with reasons, videos waiting for the user — and stop the schedule.
