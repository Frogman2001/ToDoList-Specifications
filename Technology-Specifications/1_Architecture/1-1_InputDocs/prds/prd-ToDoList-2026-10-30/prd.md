---
title: ToDoList
status: final
created: 2026-10-30
updated: 2026-10-30
---

# PRD: ToDoList (Project 1)
*Working title — confirm.*

## 0. Document Purpose

This PRD defines project 1 of ToDoList for the author (product owner and sole developer) and for the downstream workflows: UX, architecture, epics, and build. It builds on the product brief (`_bmad-output/planning-artifacts/briefs/brief-ToDoList-2026-10-30/brief.md`). Glossary terms are used exactly. Technology intent lives in `addendum.md`.

## 1. Vision

ToDoList is a date-driven personal task app, on web and Android, where you add things fast and see what's on your plate for today and this week.

## 2. Target User

The primary user is the author. Family and a few others may use it later, each seeing only their own items.

### 2.1 Jobs To Be Done

- **Quick capture (most important).** When I am at work or away from home, I want to open the app and add an item as simply as writing a note on a Post-it, so I don't lose it. I often add several in a row.
- **Glance.** When I am away from home, I want to look at everything for today, so I know what I have to do later.
- **Plan and work.** When I get home, I want to open the web app, put the day's items in order, and start working through them. Items default to creation order, and I can reorder them by hand.
- **Check on the move.** When I leave the house for an errand, I want to see the current state of today's items.

### 2.2 Key User Journeys

- **UJ-1. Russ captures things between meetings.**
  - **Persona + context:** Russ is at work with his phone, and thoughts keep arriving.
  - **Entry state:** signed in on Android.
  - **Path:** opens the app, types a description, and adds it. Types another and adds it. Does it a third time.
  - **Climax:** each item is captured in seconds with no list to pick, and it appears on today's view.
  - **Resolution:** later he opens the app and glances at today's items to see what is waiting.

- **UJ-2. Russ plans his evening at home.**
  - **Persona + context:** Russ gets home and wants to organize the day before working.
  - **Entry state:** signed in on the web, at a full keyboard.
  - **Path:** opens today's view, sees what he captured plus anything overdue, puts the items in order, and starts working them. He adds more if something occurs to him. He may mark items in progress or done, or push some to tomorrow.
  - **Climax:** today's list is in the order he wants, and he can start working.
  - **Resolution:** before he leaves for an errand, he checks today's state on his phone.

Web and mobile offer the same functionality; neither is read-only.

## 3. Glossary

- **Item** — One task. Has a description, state, detail, due date, completed date, and a position in order. Belongs to exactly one user.
- **Description** — The short text of an item. The only field required to add one.
- **Detail** — Optional longer notes on an item.
- **State** — One of not started, in progress, or done.
- **Due date** — The date an item is planned for. Required in data, and defaults to today. Changes only when the user changes it. Drives all display.
- **Completed date** — The date an item became done. Stored for information; not used for display.
- **View** — A computed display of items: Today, a specific date, or a week. There are no stored lists.
- **Today view** — Shows every not-started and in-progress item due today or earlier, plus done items due today.
- **Overdue item** — A not-started or in-progress item with a due date before today.
- **Add control** — Where a user types a description to add an item. The Today view and the specific-date view have one; the week view has one per day.
- **Reschedule** — Changing an item's due date: pick a date, or push to tomorrow, next week, or next month.
- **Position** — An item's place in order among the items with the same due date. Default is creation order. Moving an item to another date puts it at the end of that date.
- **Week view** — The items for each day of a week, with a divider between days, earliest day first (leftmost when shown as columns).

## 4. Features

### 4.1 Sign in
**Description:** The user signs in with Google. Each user sees only their own items.

#### FR-1: Sign in with Google
A person can sign in with a Google account, and the app creates their user on first sign-in. Realizes UJ-1, UJ-2.
- A signed-out visitor sees only the sign-in prompt.
- A signed-in user sees only their own items.

#### FR-2: Sign out
A user can sign out. After signing out, no items are shown.

### 4.2 Quick add
**Description:** Adding an item is as simple as writing a note on a Post-it. Only a description is needed. The add control is ready as soon as the app opens, and stays ready after each add, so several items can go in one after another. Realizes UJ-1, UJ-2.

#### FR-3: Add an item with a description only
A user can add an item by entering a description and submitting. The item gets state not started, and goes to the end of its date. The date is today on the Today view, the viewed date on a specific-date view, and the day it was added in on a week view, where each day has its own add control.
- A description is the only required entry. Detail and state may also be set while adding; state defaults to not started.
- The new item appears in the current view straight away.
- After submitting, the add control is empty and ready for the next item.
- An empty description is not added.

### 4.3 Views
**Description:** Views are computed from due dates; no lists are stored. Items sort by due date ascending, then position ascending within each date.

#### FR-4: Today view
A user can see the Today view. Realizes UJ-1, UJ-2.
- Shows every not-started and in-progress item due today or earlier, and done items due today.
- Overdue items appear before today's items, oldest due date first.
- "Today" is the date on the user's device: the phone's date on Android, the browser's date on the web.

#### FR-5: Specific-date view
A user can pick a date and see the items due on it, including done items. This is how past days are reviewed.

#### FR-6: Week view
A user can see a week and move to other weeks. A week runs Monday to Sunday. It shows all items with a due date in that week, with dividers between days, earliest day first, and each day has its own add control.
- Laid out as a vertical list with dividers, or as one column per day, depending on screen size. [NOTE FOR PM: layout choice belongs to UX.]

#### FR-7: Hide completed
A user can hide done items in any view, and show them again.

### 4.4 Edit, state, and delete

#### FR-8: Edit an item
A user can edit an item's description, detail, and due date.
- A date can be chosen from a calendar picker or typed. Typed dates accept digits only (punctuation is added for the user), or digits separated by slashes or dashes.

#### FR-9: Change state
A user can set an item to not started, in progress, or done.
- Setting done records the completed date.
- Setting done does not change the due date.
- Moving an item out of done clears the completed date. [ASSUMPTION]

#### FR-10: Delete an item
A user can delete an item after confirming. Cancelling leaves the item unchanged.

### 4.5 Reschedule

#### FR-11: Reschedule an item
A user can change an item's due date by picking a date, or by pushing it to tomorrow, next week, or next month, each as a single action. Realizes UJ-2.
- The item goes to the end of the target date's items.
- The item leaves its current position without changing any other item's due date.
- "Next week" is the following Monday, and "next month" is the first of the next month.

### 4.6 Reorder

#### FR-12: Reorder by dragging
A user can drag an item to change its position among the items with the same due date, in any view, including done items. Realizes UJ-2.
- The new order persists and is the same on web and mobile.
- Dragging an item across a day divider reschedules it to that day. [ASSUMPTION: it lands where it is dropped; in the Today view, dragging an overdue item into today's items sets its due date to today.]

### 4.7 Parity

#### FR-13: Same functionality on web and mobile
Every requirement above works on both the web app and the Android app, except FR-15, which is web only.

#### FR-14: Opening view
When the app opens, the web app shows the week view containing today, and the Android app shows its Add new screen. Realizes UJ-1, UJ-2.

#### FR-15: Move several items (web only)
A web user can select several items and move them to another date together. Only moving is available for several items at once; other changes are made one item at a time.
- Moving several items uses the same rules as FR-11.

### 4.8 Cross-cutting requirements

- **NFR-1: Data isolation.** A user can never see or change another user's items.
- **NFR-2: Sign-in required.** No item data is available without signing in.
- **NFR-3: Consistent results.** For the same items and the same date, the web app and the Android app show the same views and the same order.

## 5. Non-Goals

- Not a Trello replacement. No boards, lists, labels, checklists, or recurring items.
- No sharing between users. Each user sees only their own items.
- No offline mode.
- No address field. Addresses live on calendar entries, which are separate from this app.
- No admin tools, and no public promotion until admin tools exist.
- Not designed for scale before it is a real problem.

## 6. Scope

### 6.1 In Scope (Project 1)
- Everything in §4, on both the web app and the Android app, deployed to production.
- Sign in with Google only.

### 6.2 Out of Scope (Project 1)
- Everything in §5, plus other sign-in providers, an iOS app, and display by completed date. Any of these can be picked up in a later project.

## 7. Success Metrics

**Primary**
- **SM-1**: Delivery. Every requirement in §4 (FR-1 to FR-15) is live in production, on both the web app and the Android app except FR-15 (web only). Validates FR-1 to FR-15.
- **SM-2**: Real use. The author uses the app on most days for a month and has not gone back to Trello for daily planning. Validates FR-3, FR-4, FR-11.

## 8. Open Questions

None blocking. See the Assumptions Index.

## 9. Assumptions Index

- §4.4 FR-9: moving an item out of done clears its completed date.
- §4.6 FR-12: a dragged item lands where it is dropped; in the Today view, dragging an overdue item into today's items sets its due date to today.
