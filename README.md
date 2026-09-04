# screenshot-to-calendar

**In short:** screenshot something on your phone — a flyer for an event, a
scholarship post, an appointment — share it to Claude and add the tag
`/ss-info`. Claude reads the screenshot, figures out what it is, and puts it
on the right Google Calendar for you. No typing it in yourself.

- 📸 See something worth remembering → screenshot it.
- 🕹️ Open Control Center and tap the `ss-info` shortcut (one tap, see
  setup below for how to add it there).
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
        │  tap the shortcut in Control Center
        ▼
  An iOS Shortcut ("ss-info")
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

**Get the Shortcut:**

- **Download it:** [`shortcuts/ss-info.shortcut`](shortcuts/ss-info.shortcut)
  — open that link on an iPhone (or AirDrop/save the file to your phone) and
  tap it; iOS opens the Shortcuts app's import screen. Review the actions it
  shows before adding it, same as with any shortcut you didn't build
  yourself.
- **Or build your own** in the Shortcuts app: an action that grabs your most
  recent screenshot, tags it with `/ss-info`, and hands it to the Claude app
  (Shortcuts app → **+** → new shortcut → add a "Get Latest Screenshots"-type
  action, then a **Share** action targeting **Claude**).

**Add it to Control Center (one-time):**
1. Open the **Settings** app → **Control Center**.
2. Tap **Add a Control**.
3. Find **Shortcut** in the list, tap it, then choose `ss-info` from your
   shortcuts.
4. Drag it into position if you want it somewhere specific, then close
   Settings.

(On older iOS versions that don't support adding an individual shortcut as
its own control, add the general **Shortcuts** control instead — tapping it
in Control Center opens your shortcuts list, where you tap `ss-info`.)

**Using it:**
1. Take a screenshot of whatever you want to save (a flyer, a scholarship
   post, an appointment).
2. Swipe down from the top-right corner to open Control Center, and tap the
   `ss-info` icon.
3. That's it — the shortcut grabs the screenshot, tags it `/ss-info`, and
   sends it to Claude on its own. No need to open the app, find the
   screenshot, or type anything.
4. Claude reports back once the event is already on the calendar.

If you'd rather trigger it from the Share Sheet instead of Control Center
(e.g. to forward something that isn't a screenshot), enable **Show in Share
Sheet** in the shortcut's settings and share to it the normal way — you'll
just need to type `/ss-info` yourself in that case, since the Share Sheet
path doesn't auto-tag the message the way the Control Center trigger does.

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
3. Duplicate the same iPhone Shortcut in the Shortcuts app, edit its
   **Share** action to target **ChatGPT** instead of Claude, and add that
   copy to Control Center the same way (see Option A above).
4. Since ChatGPT's share extension doesn't auto-tag the message, you'll
   need to type/paste `/ss-info` yourself once ChatGPT opens with the image.

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
shortcuts/ss-info.shortcut  — the iPhone Shortcut, ready to import
```

To change behavior (add a category, change reminder rules, point at a
different calendar), edit `skills/ss-info/SKILL.md` directly — that file
*is* the skill, not just documentation of it.
