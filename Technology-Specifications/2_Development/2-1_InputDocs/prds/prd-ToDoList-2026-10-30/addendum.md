# Addendum: ToDoList PRD (Project 1)

Technical-how and rationale that does not belong in the PRD's requirements.

## Technology intent (decided in architecture)

- **Mobile:** .NET MAUI on Android. iOS is not in project 1. MAUI remains a learning goal.
- **Web:** React.
- **Hosting:** Azure. The API is a Function App that scales to zero when idle.
- **Shape:** one back end, two front ends (web and Android). Both clients call the API.

## Data store intent

- Cosmos DB, on the free tier (provisioned throughput). An account document holds the user and future account settings. Each item is its own document, so completed items do not accumulate inside one document. Each document has a schema version: reads translate older versions to the current one, and saves write the current version. A save sends the document's Cosmos etag so a stale write is rejected. The client snaps back and reloads, for an account document and for an item.
- Position is per due date, so reordering and rescheduling only touch the affected dates' items.

## Field limits and errors (decided in architecture)

- Description is required, trimmed, 1 to 200 characters. Detail is optional, up to 2,000 characters. Dates cross the API as `YYYY-MM-DD`. Those rules live in one shared contract. Each client checks them before sending, and the API checks them again.
- A failed request returns problem-details. A validation failure names the field. A stale etag is a conflict, and the client snaps back and reloads. A server failure or no response uses the words `Error reaching server`. The body does not include exception text.

## Implementation guidance (from the author; not requirements)

- **Week start day:** Monday, held in one constant or config value exposed through a property, so it can become user-configurable later.
- **Day component:** one reusable block of code for a "day" (header, items, add control). The date view renders one; the week view renders seven. The author wants all days in the week view to look the same, but has not thought it through in detail.

## Date handling (for architecture)

- "Today" comes from the client device (phone date on Android, browser date on web), so the back end should store due and completed dates as plain calendar dates with no time zone, and the clients should compute Today, week, and ordering the same way.
- While the app is open, Today stays as loaded. The client recomputes Today when the app returns to the foreground. Hide completed is stored on the device, with no server call.
