---
title: "Product Brief: ToDoList"
status: complete
created: 2026-10-30
updated: 2026-10-30
---

# Product Brief: ToDoList

## Executive Summary

A personal, intentionally simple to-do app (web + mobile) built around *when*: what am I doing today, this week, next week? Inspired by Trello, but without the accumulated features that annoy the author. Built and maintained as a hobby, to professional engineering standards, first for the author, then for family and a few others.

The work is delivered as a series of bounded projects, each with a defined end, like sprints or quarterly projects. This brief covers **project 1**. Later projects get decided after living with the result.

## The Problem

Trello's accumulated features annoy the author; it has grown more complex than the daily "what's my plan" job needs. Secondary problem: the author needs a real project application to build.

## The Solution

**Guiding pattern:** data is stored plainly; the display interprets it. Lists are not a first-class concept. Every item has a due date (required in data, defaulting to today if the user omits it), and views are computed from it.

- **Views:** today, a specific date, this week, next week, or another week. A week view shows every item with a due date in that week.
- **Today** shows every not-started or in-progress item due today or earlier, so nothing overdue gets lost. Done items do not roll forward. They stay on their own date.
- **Two primary actions on open:** add an item to the current view, or look at the current view.
- **Reschedule in one action:** set a specific date, or push to tomorrow, next week, or next month. A due date changes only when the user changes it.
- **Item fields:** description, state (not started / in progress / done), detail, due date, completed date. Display is driven by due date. Completed date is informational for now.
- **Done items stay visible**, with a "hide completed" toggle. Scrolling back through past dates shows what was done on each date.
- **Reorder** items by dragging. Items sort by due date, then by position within each date, and both front ends offer the same functionality.
- **Delete** an item, with a standard confirmation.
- **One back end, two front ends:** web (full keyboard) and mobile (on the go).

## What Makes This Different

Not "fewer features" alone. Three things:
- **No lists.** Everything is a dated item, and views are computed.
- **Overdue items roll into Today** until done.
- **One-action rescheduling.**

## Who This Serves

- **Primary:** the author, for daily personal organization.
- **Also:** family and a few other people, each seeing only their own items. Not publicized until admin tools exist.

## Success Criteria

Project 1, as scoped below, is complete and deployed to production. Building and maintaining the app is itself the point (hobby and professional development).

## Scope

**Project 1 (in):**
- Sign in with Google.
- Add, edit, reorder (by dragging), and delete items (with confirmation), and change state.
- Views: today, a specific date, and weekly.
- Reschedule: pick a date, tomorrow, next week, next month.
- Hide completed.
- Web front end and mobile front end, one shared back end.
- Deployed to production.

**Out of project 1:**
- Offline mode (too much complexity for now).
- Sharing between users.
- Address field (lives on calendar entries, which are separate from this app).
- Other SSO providers (Facebook, etc.).
- Admin tools and public promotion.
- Anything not above: built only when needed.

**Principles:**
- Features stay simple and usable.
- Implementation is clean and readable, and follows common coding and testing practices.
- Startup mindset: do not build what is not needed, and do not design for scale before it is a real problem.
- It is a hobby, so perfection is not expected early.
- No formal feature cap. The author removes anything that annoys them.

## Technical Direction (intent, not decided)

Azure hosting is intended, not required. One back end serves two separate front ends, a deliberate engineering challenge. Detail belongs in the architecture.

## Vision

A small, dependable app that the author, family, and a few others use every day. If it ever outgrows that, the author will decide then whether to scale or rebuild. That would be a good problem to have, not something to design for now.
