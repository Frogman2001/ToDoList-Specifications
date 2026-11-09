# Addendum: ToDoList PRD (Project 1)

Technical-how and rationale that does not belong in the PRD's requirements.

## Technology intent (not decided; for architecture)

- **Mobile:** Android is required. .NET MAUI is the intended framework: the author has never built with it, and it may allow an iOS app later. Treat as a learning goal alongside a product goal.
- **Hosting:** Azure intended, not required.
- **Shape:** one back end, two front ends (web and mobile).

## Data store intent (not decided; for architecture)

- Document database. Proposed shape: a User document, and a document for the user's items. Architecture to decide whether items are embedded in one document or stored as separate documents (growth, per-item reorder writes, and query by due date matter here).
- Position is per due date, so reordering and rescheduling only touch the affected dates' items.

## Implementation guidance (from the author; not requirements)

- **Week start day:** Monday, held in one constant or config value exposed through a property, so it can become user-configurable later.
- **Day component:** one reusable block of code for a "day" (header, items, add control). The date view renders one; the week view renders seven. The author wants all days in the week view to look the same, but has not thought it through in detail.

## Date handling (for architecture)

- "Today" comes from the client device (phone date on Android, browser date on web), so the back end should store due and completed dates as plain calendar dates with no time zone, and the clients should compute Today, week, and ordering the same way.
- Midnight rollover while the app is open is not specified; architecture to decide.
