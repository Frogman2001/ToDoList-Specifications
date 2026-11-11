# Addendum: ToDoList Product Brief (Project 1)

Context that belongs in downstream documents (PRD, architecture) or that did not fit the brief.

## Rejected or removed

- **Address field.** Addresses are better stored on calendar entries, which the author already uses and which are separate from this app.
- **Feature cap or "never" list.** Rejected. The author is the only developer, trusts themselves, and will remove any feature that annoys them.
- **Offline mode.** Excluded from project 1. It adds a lot of complexity that is not needed yet.
- **Facebook and other SSO providers.** Deferred. Project 1 supports Google only.
- **Lists or buckets as a data concept.** Rejected in favour of computed views over dated items. Today, Tomorrow, and Future began as buckets and became views.

## Date model detail (for the PRD)

The brief states the core rules. Details beyond them:

- The **completed date** is set when state becomes done. Completing an item does not change its due date.
- Done items appear on Today only if they are due today.
- The user can move between weeks and between individual dates.
- Both dates exist because they are different concepts, though they are usually the same day.

## Parked for later projects

- Admin tools (needed before any public promotion).
- Displaying or filtering by completed date.
- What scaling or a rebuild would look like if usage grew.
- Additional sign-in providers.
- Sharing between users, and offline use.

## Delivery model

Work is delivered as sequential, bounded projects, like sprints or quarterly projects. Each has a defined end. The author decides what to build next after living with the result. Each project follows the standard BMAD flow with scope limited to that project.
