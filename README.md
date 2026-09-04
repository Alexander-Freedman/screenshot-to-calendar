# screenshot-to-calendar

Screenshot something on your phone — an Instagram flyer for a club meeting, a
scholarship post, a doctor's office text, whatever — send it to Claude tagged
`/ss-info`, and it lands on the correct Google Calendar with no manual typing.
This repo documents the whole pipeline: the iPhone side, the Claude side, and
the calendar side.

```
Instagram / Photos / any screenshot
        │  Share Sheet
        ▼
  iOS Shortcut "ss-info"  (shortcuts/ss-info.shortcut)
        │  opens Claude (or ChatGPT) with the image + "/ss-info" tag
        ▼
  Claude reads the image, decides what it is
        │  runs the ss-info skill (skills/ss-info/SKILL.md)
        ▼
  Google Calendar (via the Google Calendar connector/MCP tool)
        │  Google Calendar:create_event
        ▼
  Event appears on the right calendar automatically
```

> **Note on the uploaded `.shortcut` file:** Apple exports Shortcuts as a
> *signed* binary — everything except the certificate chain is
> encrypted/compressed, so it can't be inspected as text (a `strings` dump
> only turns up Apple's OCSP/CRL URLs, nothing about the actions inside). The
> setup instructions below describe how to build/use the equivalent Shortcut
> from scratch in the Shortcuts app, since the exact action list in the
> attached file can't be read from outside Shortcuts. `shortcuts/ss-info.shortcut`
> is kept in this repo as a backup of the original export — open it on an
> iPhone (AirDrop, Files, or the Shortcuts app) to install/inspect it directly.

## 1. What actually happens on the calendar side

Claude doesn't get to freehand this — the skill hard-codes exactly where
things go. There are four real categories, each mapped to one of Alex's
actual Google Calendars (confirmed live via `Google Calendar:list_calendars`):

| What was sent | Calendar | Event type |
|---|---|---|
| Club event / meeting | **Clubs** | Timed event, real date/time + location. No reminder. |
| Club officer application | **Clubs** | All-day event on the deadline, **one** reminder |
| Scholarship / internship opportunity | **Internships & Opportunities** | All-day event on the deadline, **one** reminder |
| Doctor's appointment / medical | **Appointments** | Timed event, real date/time + location. No reminder. |
| Anything else / genuinely unclear | — | Claude asks Alex instead of guessing |

Everything else in the Google account (Class, Research, Tutoring,
Professional Events, Personal, etc.) is intentionally **out of scope** for
this flow — the skill is written to only ever touch those three calendars,
by their exact names, never the primary/default calendar.

Two rules that matter more than they look:

- **Only the real Google Calendar tool is used** — `Google Calendar:create_event`,
  the actual Google Calendar API call. iPhone/Claude environments can also
  expose a generic, phone-native calendar action (sometimes shown as
  `event_create_v0`/`event_create_v1`, with no "Google Calendar:" prefix) —
  that one writes to the **Apple Calendar app**, not Google Calendar, and the
  skill explicitly refuses to use it even if it's offered as the easier
  default.
- **Deadlines get exactly one reminder; timed events get none.** A club
  meeting or doctor's appointment just needs to exist on the calendar. A
  scholarship or officer-application deadline is worth a single, sensibly
  timed nudge — never a stack of default reminders.

If a screenshot contains multiple dates (a full rush-week schedule, a
recurring series), Claude creates one event per date, skipping anything
already in the past.

## 2. The Claude skill

The logic above lives in [`skills/ss-info/SKILL.md`](skills/ss-info/SKILL.md).
It's a Claude **Skill** — a standing instruction set attached to Alex's
Claude account, not something rebuilt per-conversation. It triggers on the
literal text `/ss-info`, whether that's followed by an image, plain text, or
both.

Behaviorally, the skill is deliberately biased toward *acting*, not asking:

- A missing/TBD location never blocks creating the event.
- A date that's incomplete but reasonably inferable ("Wednesday at 7pm" with
  context making the date obvious) still gets added — Claude just says what
  it inferred.
- A date that's actually jumbled or contradictory (bad OCR, conflicting
  times) is the one case Claude stops and asks about, rather than inventing
  something plausible.
- Claude never asks "do you actually want this on your calendar?" — sending
  it tagged `/ss-info` *is* that confirmation. It reports back only **after**
  the event already exists, not as a pre-approval step.

For scholarships/internships specifically, the skill also cross-checks
Alex's existing scholarship search notes (no need-based awards, no
sweepstakes/lead-gen, known eligibility constraints) and gives a one-line
fit read — but still adds the reminder unless the opportunity is clearly
excluded by those existing rules.

## 3. Setting it up on iPhone

### Option A — Claude app (this is what the skill is built for)

This is the fully-automatic path: the skill and the Google Calendar
connector both live on Alex's Claude account, so anything sent this way
needs zero manual calendar work.

**One-time setup:**
1. In Claude's web/app settings, make sure the **ss-info** skill is
   installed/enabled on the account.
2. Under **Connectors**, connect **Google Calendar** (OAuth sign-in) and
   grant it access to the calendar. This connector is what exposes the
   `Google Calendar:list_calendars` / `Google Calendar:create_event` tools
   the skill calls.
3. Install the **Claude** app on iPhone and sign into the same account.

**Build the Shortcut (Shortcuts app):**
1. Shortcuts app → **+** → new shortcut.
2. Add **Receive [Images or Text] from Share Sheet** as the first action, so
   it can run on both a screenshot and on plain relayed text.
3. Add a **Share** action, and set its destination to the **Claude** app.
   Sharing to Claude opens a new chat with the shared image (or text)
   already attached.
4. Rename the shortcut `ss-info`, and in its settings enable **Show in Share
   Sheet** (restrict accepted types to Images + Text).

iOS sandboxes what a Shortcut can do inside another app's compose box, so
the Shortcut can hand the image/text off to Claude, but can't also type and
send the message for you. In practice that's a two-second step:

**Using it:**
1. On an Instagram post (or any screenshot in Photos), tap **Share**.
2. Pick **ss-info** from the share sheet — this hands the image to Claude in
   a new chat.
3. Type `/ss-info` and send. The skill takes it from there and reports back
   once the event is already on the calendar.

If what you're forwarding is already text (e.g. relayed from some other
automation that already OCR'd a screenshot), the same shortcut/flow works —
just make sure `/ss-info` is part of the message.

### Option B — ChatGPT app (manual equivalent, no native skill support)

ChatGPT has no equivalent to a Claude Skill that auto-triggers on a tag, so
this path is a hand-built approximation rather than a real port:

1. Create a **Custom GPT** (e.g. named "ss-info") and paste the contents of
   [`skills/ss-info/SKILL.md`](skills/ss-info/SKILL.md) into its instructions,
   swapping the tool names for whatever calendar-writing action the GPT
   actually has available.
2. Give that GPT a **Google Calendar** action/connector with write access,
   and hard-code the exact calendar names — **Clubs**, **Internships &
   Opportunities**, **Appointments** — into the instructions, since there's
   no guarantee the GPT will resolve calendar names the same safe way the
   Claude tool does.
3. Reuse the same iPhone Shortcut, just pointing the **Share** action at
   **ChatGPT** instead of Claude.
4. Same manual step: type/paste `/ss-info` once ChatGPT opens with the image.

Treat this path as best-effort. The categorization and safety rules only
really exist as enforced behavior inside the Claude skill; the ChatGPT
version is only as reliable as the custom instructions you paste in, and
needs to be kept in sync by hand if `SKILL.md` changes.

## 4. Troubleshooting

- **Nothing happens** — the message has to contain the literal `/ss-info`
  text. An image alone only triggers the skill if it's unambiguously one of
  the four categories.
- **Claude asks a question instead of just adding it** — expected when the
  date/time is actually contradictory or the category is genuinely
  ambiguous. Answer it; the skill won't guess in that situation.
- **Event landed on the wrong calendar / on Apple Calendar instead of
  Google** — that's the exact failure mode the skill's tool-selection rule
  (section 1) exists to prevent; if it happens, it means a generic calendar
  tool got used instead of `Google Calendar:create_event`.
- **A whole week's schedule only produced one event** — shouldn't happen;
  the skill is written to create one event per date. Flag it if it does.

## 5. Repo layout

```
README.md                  — this file
skills/ss-info/SKILL.md     — the actual Claude skill (source of truth for behavior)
shortcuts/ss-info.shortcut  — backup of the exported iOS Shortcut
```

To change behavior (add a category, change reminder rules, point at a
different calendar), edit `skills/ss-info/SKILL.md` directly — that file
*is* the skill, not just documentation of it.
