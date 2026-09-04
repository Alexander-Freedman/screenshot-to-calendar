---
name: ss-info
description: >
  Example Claude Skill — process something tagged "/ss-info" (a screenshot,
  typically forwarded via an iOS Shortcut, or plain text already describing
  the same thing) and add it directly to the correct Google Calendar. This is
  a template: swap the categories/calendar names below for your own before
  using it.
---

# ss-info (template)

The idea: instead of reading a screenshot yourself and manually creating a
calendar event, you forward it tagged `/ss-info` and Claude does both steps —
figure out what it is, then put it on the right calendar.

This file is a **starting point**, not a drop-in config. Before using it,
replace the example categories and calendar names below with your own.

## Categories and where each goes (example — replace with your own)

| What was sent (image or text) | Calendar | Event type |
|---|---|---|
| Event / meeting | *your "events" calendar* | Timed event — actual date/time and location. No reminder. |
| Application with a deadline | *your "deadlines" calendar* | All-day event on the deadline, **one** notification reminder (not a stack of default reminders) |
| Opportunity with a deadline (scholarship, internship, etc.) | *your "deadlines" calendar* | All-day event on the deadline, **one** notification reminder |
| Appointment | *your "appointments" calendar* | Timed event — actual date/time and location. No reminder. |
| Anything else, or genuinely unclear | — | **Ask** what it is and which calendar before creating anything |

Use your real Google Calendar names (`Google Calendar:list_calendars` will
resolve them) — don't guess a different name or fall back to the primary
calendar.

Always use the **`Google Calendar:create_event`** tool for this — the actual
Google Calendar API tool. **Never** use a generic/native calendar tool (e.g.
one tied to the phone's own Calendar app, sometimes named just
`event_create_v0`/`event_create_v1` without a "Google Calendar:" prefix) —
that writes to Apple Calendar, not Google Calendar, and is the wrong
destination even if it's offered as an easier default. If more than one tool
could plausibly create the event, confirm it's the Google Calendar one before
calling it.

If what's sent contains **multiple events** (e.g. a full week's schedule, a
multi-day series), create one calendar event per date/time listed — don't
collapse them into one event or just display them as a table without adding
them.

## Workflow

1. Read whatever was sent — an attached image, or plain text after the tag.
   Extract what it actually is, the date/time, location (if a physical
   event), and the deadline (if an application/opportunity). If it's text,
   trust it as already-extracted content rather than re-deriving it, but
   still flag anything that looks incomplete (e.g. no date given).
2. If the category is genuinely ambiguous (could be more than one type, or
   none of the listed ones), **ask**. For missing or unclear details, use
   judgment rather than a rigid rule:
   - A missing or "TBD" **location** is never a reason to hold off — create
     the event anyway with the location left blank or noted as TBD.
   - If the date/time/deadline is jumbled, contradictory, or genuinely can't
     be pinned down with confidence (garbled OCR, conflicting times,
     ambiguous which date a time belongs to) — don't guess and don't force
     an add. Flag it plainly and ask, rather than inventing a
     plausible-sounding date.
   - If the date/time is merely incomplete but reasonably inferable (e.g. a
     time with no explicit date but the context makes it clearly "this
     Wednesday"), it's fine to add with your best read — just say what you
     inferred in the report.
   - The bar is confidence, not completeness: partial-but-clear beats
     waiting for perfect information; actually-unclear beats guessing.

   Don't ask "want me to add it?" once the category and date/time are
   reasonably clear — treat the forward itself as the signal it should go on
   the calendar. If the core details are legible, just add it.
3. (Optional, project-specific) Cross-check against your own notes/rules if
   you keep any — e.g. exclusion criteria for scholarships, known
   eligibility constraints. Skip this step entirely if you don't have
   anything like that; it's not required for the skill to work.
4. Create the event(s) with the **`Google Calendar:create_event`** tool on
   the correct calendar from your table above — this step is mandatory, not
   optional, for anything that fits a category.
   - For a multi-event schedule, create each future date as its own event;
     skip any date/time that's already passed (check current time first).
   - Timed events: real date/time + location. No reminder notification
     needed.
   - Deadline events: all-day event on the deadline date itself, with a
     single appropriately-timed reminder notification attached (not
     several). Title clearly naming what it's for (e.g. `[Org] application
     due` or `[Name] deadline`).
5. Report back briefly, **after** the event is already created — what was
   added, which calendar, and when. This is a confirmation of what happened,
   not a request for permission to proceed. If anything was uncertain or
   estimated (a TBD location, a guessed time range), say so plainly and note
   it can be fixed as a follow-up edit — the event exists already, that's
   not a precondition.

## Notes

- Default to acting, not asking. A missing location never blocks the add.
  Only hold off when the core details (what it is, or when it is) are
  genuinely too jumbled/contradictory to trust.
- Don't ask for confirmation before creating the event, and don't
  re-confirm the calendar choice once a category is obvious.
- Deadline items get exactly one reminder notification, timed appropriately
  — never a stack of several. Regular timed events get no reminder
  notification at all, just the calendar entry.
