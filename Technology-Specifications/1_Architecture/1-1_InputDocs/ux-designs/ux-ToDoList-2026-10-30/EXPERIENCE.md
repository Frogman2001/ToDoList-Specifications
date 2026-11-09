---
name: ToDoList
status: final
updated: 2026-11-04
sources:
  - ../prds/prd-ToDoList-2026-10-30/prd.md
  - ../briefs/brief-ToDoList-2026-10-30/brief.md
---

# ToDoList — Experience Spine

Two peer contracts: this file owns how the app behaves, `DESIGN.md` owns how it looks. Tokens are referenced as `{path.to.token}`. Where this file and a mock disagree, this file wins. Terms (Item, View, Position, and so on) are the PRD Glossary terms.

## Foundation

Two surfaces on one back end, with the same functionality except where noted:

- **Android app** (phone, portrait and landscape; small tablet). Opens on Add new.
- **Web app** (desktop browsers at a full keyboard, and narrower windows). Opens on the week view containing today.

No UI system is named; each surface follows its platform's conventions for system gestures and text size. Dark is the first theme. Themes are swappable later on the web (architecture), so no visual value is hard-coded in behavior.

## Information Architecture

| Surface | Platform | Reached from | Purpose |
|---|---|---|---|
| Sign In | Web | Opening the app while signed out | Sign in with Google before the list is shown |
| Add new | Android | App open; leftmost column | Quick add, nothing else on screen |
| Today view | Both | Top bar link `Today` | Open items due today or earlier under date headers, then today's items |
| Day | Both | `Pick date` or scrolling | One date's items |
| Week | Both | Opening view on web; mode switch on Android | A week, Monday to Sunday, one box per day |
| Add popup | Both | The `Add` control in a day box | One form with every field: description, detail (optional), date, and state (defaults to To do). Opens in context of the list. [ASSUMPTION: on the phone it is a popup sheet that scrolls if needed] |
| Edit item | Both | Tapping or clicking an item | The same form, filled in with the item, plus reschedule choices (`Tomorrow`, `Next week`, `Next month`) and a `Delete` link. A popup on web; a full screen on the phone, so nothing sits below the fold. |
| Shortcut help | Web | `?` or top bar `Shortcuts` | List of keyboard shortcuts |

Top bar uses words only: `Previous`, `Next`, `Today`, `Hide completed`, `Shortcuts` (web), `Day` and `Week` (the mode switch on Android), `Sign In` or `Sign Out`. `Sign In` and `Sign Out` are text links at the top of the screen. On Android they appear only in the Add new top bar.

→ Layout reference: `wireframes/flow-layouts-2026-10-30.excalidraw` and `mockups/theme-a-pass-2.html`. This spine wins on conflict.

## Voice and Tone

Plain, short, no exclamation marks, no encouragement, no tours or "may I help you" prompts.

| Situation | Words |
|---|---|
| Server unreachable | `Error reaching server` |
| Empty day (web) | `Nothing here.` |
| Description box | `Add description...` |
| Delete confirmation | `Delete this item?` with `Delete` and `Cancel` |
| Add while signed out (phone) | `Sign In to save this item.` |
| Sign in, sign out | `Sign In`, `Sign Out` |

## Component Patterns

Visual specs live in `DESIGN.md`.

| Component | Behavior |
|---|---|
| Day box | One reusable block for a date: header, items in order, Add control. The Day view renders one; the Week view renders seven (eight columns on Android, counting Add new). |
| Add control | Opens the Add popup for that day, in context of the list. |
| Description box | The placeholder disappears on click (web) or when typing starts (phone; may need a tap first). Stays and clears after each add so several items can go in. Empty descriptions are not added. |
| Date field | Opens a calendar picker showing the full month, weeks starting Monday. Also accepts typing: digits only (punctuation is added), or digits separated by slashes or dashes. Echoes the understood date below the field as you type. Defaults to today, or to the day it was reached from. Month comes before day; the year defaults to the current one. |
| State box | Empty = to do, dot = in progress, check = done. On the phone, tapping it opens a choice of the three states; double-tapping the item advances the state. |
| Hide completed | Top bar link in any view. Persistence across views and restarts is left to the build. |

## State Patterns

| State | Where | Treatment |
|---|---|---|
| Loading | List views | A blank list with the state boxes cycling (empty, dot, check, repeat) until data arrives. Phone Add works at once; data loads in the background. |
| Empty day | Day, Week (web) | `Nothing here.` |
| Empty day | Phone | The description box with `Add description...` |
| Error | Any view | A lined bar at the top: `Error reaching server`, in muted red text. Clears after the next successful action. |
| Failed add | Add popup, Add new | The typed text is kept. |
| Failed move or reorder | Any view | The item snaps back to where it was. |
| Signed out | Android | Add new still opens. Pressing `Add` keeps the text and asks to Sign In, then saves. Sign-in is prompted only when needed. |
| Signed out | Web | A Sign In screen is shown before the list. |
| Picked up | Phone item | Lighter tone and an accent outline; the swipe moves it one day. |
| Selected | Web item | A thin accent outline. |

## Interaction Primitives

**Android.**
- Nothing picked up: a horizontal swipe scrolls the columns. Finger moving right reveals the column on the left (toward Add new).
- Tap and hold an item: picks up that one item. Swipe right moves it to the next day; swipe left to the previous day. The item leaves view in the two-column day mode.
- A picked-up item can be dragged up or down to reorder it, and across a day to reschedule.
- Tap an item: opens Edit item. Tap the state box: choose a state. Double-tap an item: advance state.
- Tap `Next week` or `Next month` from the item's reschedule choices, or `Tomorrow`; one item at a time.
- Picking several items is not available on the phone.

**Web, mouse.**
- Click, hold, and drag an item to move or reorder it. Dropping across a day divider reschedules it to that day, landing where it was dropped.
- Ctrl and click selects several items. The only multi-item action is move.
- Each day has an Add control that opens a popup.

**Web, keyboard.** Shortcuts that act on days and selection apply only when focus is not in a text field.

| Key | Action |
|---|---|
| Alt+N | New task |
| Enter | Save |
| Esc | Close popup or clear selection |
| Up / Down | Move selection |
| Left / Right | Previous or next day |
| Ctrl+Up / Down | Multi-select |
| Ctrl+Left / Right | Move selected item(s) one day |
| Alt+Up / Down | Reorder within the day |
| Delete | Delete (asks to confirm) |
| Space | Advance state |
| 1 / 2 / 3 | Set not started, in progress, or done |
| D / W / M | Reschedule to tomorrow, next week (following Monday), next month (the first) |
| T | Jump to today |
| ? | Show the shortcut list |

In a new item, arrow keys move between fields. Tab order follows reading order everywhere.

**Animation.** Only where it explains: an item moving, a popup arriving, and the loading cycle.

## Accessibility Floor

Common choices to start with; revise after trying it.

- Tap targets at least 48dp on Android; rows are at least 52px tall everywhere.
- Every control has an accessible name, role, and state. The description box has a hidden label, because its placeholder disappears.
- State is never conveyed by color alone; shape carries it.
- Tab order and focus order follow reading order on every surface; focus is always visible.
- Reduced motion is honored: the loading cycle shows a single static box.
- Text stays legible and controls untruncated at large system font sizes.
- Text and muted text keep sufficient contrast on `{colors.surface}`.

## Inspiration & Anti-patterns

Anti-pattern: Trello's accumulated features and its "click into a list, then find the right one" capture. This app has no lists, no labels, and no tours.

| Do | Don't |
|---|---|
| Make every action obvious and logical | Ask the user to think about where something goes |
| Words for links and buttons | Icons next to text, or icons that need guessing |
| Work with the date the user types | Demand one date format |
| Show a clear error where it can be seen | Hide errors or use alarming wording |

## Responsive & Platform

| Surface and size | Columns shown |
|---|---|
| Phone portrait, day mode | Add new and one day (one visible at a time; scroll between) |
| Phone portrait, week mode | Add new and seven days (eight columns; scroll sideways) |
| Phone landscape | Two days |
| Small tablet | Two to three days |
| Wider browsers | More columns, up to seven days for a week |

Once the column count is set, columns expand to use the full width. The Week view on the web opens on the week that contains today.

## Key Flows

Mirrors PRD UJ-1 and UJ-2.

- **UJ-1. Russ captures things between meetings.** Russ, at work with his phone, opens the app, which opens on Add new. He types a description and taps Add. The box clears, ready for the next. He adds a second and a third. **Climax:** each item is captured in seconds with no list to choose, and it is on today's view. Later he swipes to the day and glances at what is waiting. **Edge case:** if the server can't be reached, `Error reaching server` appears and his text stays in the box.
- **UJ-2. Russ plans his evening at home.** Russ, at a keyboard, opens the web app on the week view, sees what he captured plus anything overdue under its own date, and drags items into the order he wants. He presses Space to start one, `D` to push another to tomorrow, and Alt+N to add something that occurs to him. **Climax:** today's list is in the order he wants and he can start working. Before leaving for an errand he checks today's state on his phone.
