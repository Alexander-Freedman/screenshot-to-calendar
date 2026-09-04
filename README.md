# screenshot-to-calendar

**In short:** screenshot something on your phone — a flyer for an event, a
scholarship post, an appointment — share it to Claude and add the tag
`/ss-info`. Claude reads the screenshot, figures out what it is, and puts it
on the right Google Calendar for you. No typing it in yourself.

- 📸 See something worth remembering → screenshot it.
- 📤 Share it to Claude, add `/ss-info`, hit send.
- 📅 It shows up on your calendar a few seconds later, already sorted into
  the right one (events vs. deadlines vs. appointments) — Claude will only
  ask you a question if something's genuinely unclear (like a garbled date).

That's the whole thing from a user's perspective. Everything below this
point is for **setting it up** — you only need to read it once, and only if
you're the one configuring it (not needed just to use it day-to-day).

This repo is a **template**: the categories, calendar names, and workflow
are examples to copy and adapt to your own calendars, not a fixed
configuration. Nothing here is tied to a specific person's account.

---

## Setup guide (read once, then forget about it)

### How it fits together

```
Screenshot (Instagram / Photos / anywhere)
        │  Share Sheet
        ▼
  An iOS Shortcut you build ("ss-info")
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

### 1. How the calendar routing works

The skill hard-codes where things go, so Claude isn't freehanding it. The
example in this repo uses four categories — replace these with whatever
categories and calendars make sense for you:

| What was sent | Calendar | Event type |
|---|---|---|
| Event / meeting | *your "events" calendar* | Timed event, real date/time + location. No reminder. |
| Application with a deadline | *your "deadlines" calendar* | All-day event on the deadline, **one** reminder |
| Opportunity with a deadline | *your "deadlines" calendar* | All-day event on the deadline, **one** reminder |
| Appointment | *your "appointments" calendar* | Timed event, real date/time + location. No reminder. |
| Anything else / genuinely unclear | — | Claude asks instead of guessing |

Two rules that matter more than they look, regardless of what your actual
categories/calendars end up being:

- **Only the real Google Calendar tool is used** — `Google Calendar:create_event`,
  the actual Google Calendar API call. Some environments also expose a
  generic, phone-native calendar action (sometimes shown as
  `event_create_v0`/`event_create_v1`, with no "Google Calendar:" prefix) —
  that one writes to the **Apple Calendar app**, not Google Calendar. The
  skill should always confirm which tool it's calling before using it if
  more than one is offered.
- **Deadlines get exactly one reminder; timed events get none.** A meeting
  or appointment just needs to exist on the calendar. A deadline is worth a
  single, sensibly timed nudge — never a stack of default reminders.

If a screenshot contains multiple dates (a full week's schedule, a
recurring series), the skill creates one event per date, skipping anything
already in the past.

### 2. The Claude skill

The logic above lives in [`skills/ss-info/SKILL.md`](skills/ss-info/SKILL.md)
as a template. A Claude **Skill** is a standing instruction set attached to
your account — not something rebuilt per-conversation. It triggers on the
literal text `/ss-info`, whether that's followed by an image, plain text, or
both.

Before using it, edit the file to:
1. Replace the example calendar names with your real Google Calendar names.
2. Replace/trim the categories to match what you actually want routed
   automatically.
3. Drop or adapt the optional cross-check step (step 3 in the workflow) —
   it's a placeholder for any personal filtering rules you want to apply
   before adding certain event types; skip it if you don't need it.

Behaviorally, the skill is written to be biased toward *acting*, not asking:

- A missing/TBD location never blocks creating the event.
- A date that's incomplete but reasonably inferable still gets added — the
  report just says what was inferred.
- A date that's actually jumbled or contradictory (bad OCR, conflicting
  times) is the one case where it stops and asks, rather than inventing
  something plausible.
- It doesn't ask "do you actually want this on your calendar?" — sending it
  tagged `/ss-info` is treated as that confirmation. It reports back only
  **after** the event already exists, not as a pre-approval step.

### 3. Setting it up on iPhone

#### Option A — Claude app (this is what the skill is built for)

This is the fully-automatic path: once the skill and the Google Calendar
connector are both set up on your Claude account, anything sent this way
needs zero manual calendar work.

**One-time setup:**
1. In Claude's settings, install/enable a skill using
   `skills/ss-info/SKILL.md` as your starting point (edited with your own
   calendars first).
2. Under **Connectors**, connect **Google Calendar** (OAuth sign-in) and
   grant it access. This connector is what exposes the
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
4. Name the shortcut (e.g. `ss-info`), and in its settings enable **Show in
   Share Sheet** (restrict accepted types to Images + Text).

iOS sandboxes what a Shortcut can do inside another app's compose box, so
the Shortcut can hand the image/text off to Claude, but can't also type and
send the message for you — that's one extra tap.

**Using it:**
1. On a post or screenshot, tap **Share**.
2. Pick your shortcut — this hands the image to Claude in a new chat.
3. Type `/ss-info` and send. The skill reports back once the event is
   already on the calendar.

If what you're forwarding is already text (e.g. relayed from some other
automation that already extracted a screenshot's contents), the same flow
works — just make sure `/ss-info` is part of the message.

#### Option B — ChatGPT app (manual equivalent, no native skill support)

ChatGPT has no equivalent to a Claude Skill that auto-triggers on a tag, so
this path is a hand-built approximation rather than a real port:

1. Create a **Custom GPT** and paste an adapted version of
   [`skills/ss-info/SKILL.md`](skills/ss-info/SKILL.md) into its
   instructions, swapping the tool names for whatever calendar-writing
   action the GPT actually has available.
2. Give that GPT a **Google Calendar** action/connector with write access,
   and hard-code your exact calendar names into the instructions, since
   there's no guarantee the GPT resolves calendar names the same safe way
   the Claude tool does.
3. Reuse the same iPhone Shortcut, just pointing the **Share** action at
   **ChatGPT** instead of Claude.
4. Same manual step: type/paste `/ss-info` once ChatGPT opens with the
   image.

Treat this path as best-effort — it's only as reliable as the custom
instructions pasted in, and needs to be kept in sync by hand if the skill
template changes.

### 4. Troubleshooting

- **Nothing happens** — the message has to contain the literal `/ss-info`
  text. An image alone only triggers the skill if it's unambiguously one of
  your configured categories.
- **Claude asks a question instead of just adding it** — expected when the
  date/time is actually contradictory or the category is genuinely
  ambiguous. Answer it; the skill won't guess in that situation.
- **Event landed on the wrong calendar / on Apple Calendar instead of
  Google** — that's the exact failure mode the skill's tool-selection rule
  (section 1) exists to prevent; if it happens, a generic calendar tool got
  used instead of `Google Calendar:create_event`.
- **A whole week's schedule only produced one event** — shouldn't happen;
  the skill is written to create one event per date.

### 5. Repo layout

```
README.md                  — this file
skills/ss-info/SKILL.md     — the Claude skill template (source of truth for behavior)
```

To change behavior (add a category, change reminder rules, point at a
different calendar), edit `skills/ss-info/SKILL.md` directly — that file
*is* the skill, not just documentation of it.
