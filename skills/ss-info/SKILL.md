---
name: ss-info
description: Process something Alex sends tagged "/ss-info" — either a screenshot (typically from Instagram, via his iOS Shortcut) or plain text (e.g. a description relayed from an automated pipeline that already converted a screenshot to text) — describing a club event, a club officer application, a scholarship/internship opportunity, a doctor's appointment, or something else entirely — and add it directly to the correct Google Calendar. Trigger on the literal text "/ss-info" whether followed by text content or an attached image, and also on an attached image with little/no other text when it's clearly one of these categories. If the category or a key detail (date, time, deadline) isn't clear from what was sent, ask Alex rather than guessing or silently skipping it.
---

# ss-info

Alex screenshots things (mostly Instagram posts) and forwards them tagged "/ss-info" instead of manually reading each one and adding it to the right calendar himself. The tag may come with an actual screenshot attached, or with plain text describing the screenshot's contents (e.g. relayed from an automated pipeline that already extracted the details) — treat both the same way.

## Categories and where each goes

| What was sent (image or text) | Calendar | Event type |
|---|---|---|
| Club event / meeting | **Clubs** | Timed event — use the actual date/time and location shown. No reminder notification. |
| Club officer application | **Clubs** | All-day event on the deadline, with **one** notification reminder set appropriately (not a stack of default reminders) |
| Scholarship or internship opportunity | **Internships & Opportunities** | All-day event on the deadline, with **one** notification reminder set appropriately (not a stack of default reminders) |
| Doctor's appointment / medical | **Appointments** | Timed event — use the actual date/time and location shown. No reminder notification. |
| Anything else, or genuinely unclear which of the above it is | — | **Ask Alex** what it is and which calendar before creating anything |

These are Alex's real **Google Calendar** calendars (`Google Calendar:list_calendars` will resolve them) — use them exactly, don't guess a different name or fall back to the primary calendar. Always use the **`Google Calendar:create_event`** tool for this — this is the actual Google Calendar API tool, not a generic "create event" action. **Never** use a generic/native calendar tool (e.g. one tied to the phone's own Calendar app, sometimes named just `event_create_v0`/`event_create_v1` without a "Google Calendar:" prefix) — that writes to Apple Calendar, not Google Calendar, and is the wrong destination even if it's offered as an easier default. If more than one tool could plausibly create the event, confirm it's the Google Calendar one before calling it.

If what's sent contains **multiple events** (e.g. a full rush week schedule, a multi-day event series), create one calendar event per date/time listed — don't collapse them into one event or just display them as a table without adding them.

## Workflow

1. Read whatever was sent — an attached image, or plain text after the tag. Extract what it actually is, the date/time, location (if a physical event), and the deadline (if an opportunity/application). If it's text, trust it as already-extracted content rather than re-deriving it, but still flag anything that looks incomplete (e.g. no date given).
2. If the category is genuinely ambiguous (could be more than one type, or none of the listed ones), **ask Alex**. For missing or unclear details, use judgment rather than a rigid rule:
   - A missing or "TBD" **location** is never a reason to hold off — create the event anyway with the location left blank or noted as TBD.
   - If the date/time/deadline is jumbled, contradictory, or genuinely can't be pinned down with confidence (garbled OCR, conflicting times, ambiguous which date a time belongs to) — don't guess and don't force an add. Flag it plainly and ask, rather than inventing a plausible-sounding date.
   - If the date/time is merely incomplete but reasonably inferable (e.g. a time with no explicit date but the context makes it clearly "this Wednesday"), it's fine to add with your best read — just say what you inferred in the report.
   - The bar is confidence, not completeness: partial-but-clear beats waiting for perfect information; actually-unclear beats guessing.
   
   Do **not** ask whether he's "actually interested," whether something "fits his major/track," or whether he wants it saved as a tracked interest — that's not this skill's call to make and isn't a reason to withhold the calendar add. Do **not** ask "want me to add it?" once the category and date/time are reasonably clear — that confirmation step doesn't exist in this workflow; just add it. If it's clearly a club event, scholarship, internship, officer app, or appointment, and the core details are legible, just add it — his forwarding it tagged /ss-info is the signal that he wants it on the calendar, full stop.
3. For scholarships/internships specifically: quickly cross-check against [[scholarship-internship-search-log]] and [[scholarship-search-prompt]] (no need-based awards, no sweepstakes/lead-gen, fixed eligibility constraints already known). Give Alex an honest one-line fit read, but still add the reminder unless it's clearly excluded per his existing rules — don't withhold the calendar add just because you're unsure of fit.
4. Create the event(s) with the **`Google Calendar:create_event`** tool on the correct calendar from the table above — this step is mandatory, not optional, for anything that fits a category in the table. Don't stop at describing what's in the screenshot/text; the whole point of this skill is that the calendar gets updated without Alex having to do it by hand.
   - For a multi-event schedule, create each future date as its own event; skip any date/time that's already passed (check current time first).
   - Timed events (club events, appointments): real date/time + location. No reminder notification needed — it's just on the calendar, not something to be nudged about.
   - Deadline events (opportunities, officer apps): all-day event on the deadline date itself, with a single appropriately-timed reminder notification attached (not several) — these are the ones worth a nudge for. Title clearly naming what it's for (e.g. `[Org] officer app due` or `[Scholarship name] deadline`).
5. Report back briefly, **after** the event is already created — what was added, which calendar, and when. This is a confirmation of what happened, not a request for permission to proceed. If anything was uncertain or estimated (a TBD location, a guessed time range), say so plainly in that same report and note Alex can just tell you to fix it — but the event exists on the calendar already, adjustment is a follow-up edit, not a precondition. No need for a long recap of the image contents once the event's created.

## Notes

- Default to acting, not asking. A missing location (TBD or blank) never blocks the add. Only hold off when the core details (what it is, or when it is) are genuinely too jumbled/contradictory to trust — that's a judgment call, not a checklist, so don't invent a plausible date just to avoid asking, and don't ask just because something is merely incomplete rather than actually unclear.
- Don't ask for confirmation before creating the event, and don't re-confirm the calendar choice once a category is obvious — just create it and report what was done.
- Deadline items (opportunities, officer apps) get exactly one reminder notification, timed appropriately — never a stack of several. Regular timed events (club events, appointments) get no reminder notification at all, just the calendar entry.
