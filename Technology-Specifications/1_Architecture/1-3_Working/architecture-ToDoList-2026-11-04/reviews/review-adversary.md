# Adversarial review — architecture spine

**Verdict: Fail.** Two units one level down can obey every adopted Rule and still ship incompatible products. The breaks are shared-data shapes, two owners of one fact, and two legal write paths.

Target: `ARCHITECTURE-SPINE.md` (draft, 2026-11-04). Spine not modified.

## Units

Built only from the spine's Rules, the consistency table, and the structural seed. Each unit treats `Prevents` as motive and the Rule as the constraint.

- **Unit W** — `web`. No reference to Core or Scheduling (AD-11). Computes Today, week, and order on the device. Talks only to the Function App.
- **Unit A** — `ToDoList.Api`, `ToDoList.Core`, Cosmos. Only writer. Returns every item in the caller's partition. Validates `contracts/item-fields.json`, then lets Core trust the values.
- **Unit M** — `ToDoList.App` plus `ToDoList.Scheduling`. Same client obligations as W. Order and Today live in Scheduling.

Where a pair is W vs A, M makes the same client choice as W unless the finding says otherwise.

## 1. The item on the wire has no shape

**Location:** AD-5, AD-7, consistency table "Data and formats", structural seed `contracts/item-fields.json`.

**Pair:** W and A.

**Letter:**

- W sends and accepts camelCase JSON: `description`, `detail`, `state`, `dueDate`, `completedDate`, `position`, plus `id` and `etag`. It checks the AD-7 bounds before send. It never references `ItemDocumentV1`.
- A stores `ItemDocumentV1` and returns that class with the platform's default names and the Cosmos system properties: `Description`, `Detail`, `State`, `DueDate`, `CompletedDate`, `SortOrder`, `schemaVersion`, `userId`, `type`, `_etag`, `_ts`. Field limits match AD-7. `schemaVersion` is present on the stored document (AD-5). The API returns every item document (AD-1).

AD-7 names the bounds and the file. It does not name the properties, the casing, or which of `id`, `position`, `etag`, `userId`, `type`, and `schemaVersion` cross the HTTP boundary. AD-5's class names exist only in Core, which W is forbidden to reference. Both units have followed the text.

**Clash:** Every call fails to deserialize. W drops or rejects `SortOrder` and looks for `position`. A rejects or ignores `position` and `dueDate`. The same item is two structs.

**Close it — tighten AD-7 (and one sentence of AD-5):** `contracts/item-fields.json` is the HTTP schema, camelCase, and the only item properties either side may send or require:

| Property | On the wire |
| --- | --- |
| `id` | Server-assigned GUID string. Absent on create. |
| `description` | Required. Trimmed. 1–200 characters. |
| `detail` | JSON `null`, or 1–2000 characters. Empty string is a 400. |
| `state` | `not-started`, `in-progress`, or `done`. |
| `dueDate` | Required `YYYY-MM-DD`. |
| `completedDate` | JSON `null`, or `YYYY-MM-DD`. `null` exactly when `state` is not `done`. |
| `position` | Integer. See finding 5. |
| `etag` | The Cosmos `_etag` copied unchanged, quotation marks included. Present on responses and on update, batch, and delete. Absent on create. |

`userId`, `type`, `schemaVersion`, and every Cosmos system property stay off this object. Clients do not send `schemaVersion`. The API stamps the current version on save (AD-5).

## 2. Two owners of `userId`

**Location:** AD-3, AD-4, paradigm "the caller's partition".

**Pair:** W and A. A second cut of A disagrees with M.

**Letter:**

- W puts the Google `sub` it decoded into a body property `userId`, and also sends `Authorization: Bearer`. The user id is the `sub` (AD-3). The partition key is `userId` (AD-4).
- A accepts a bearer token whose audience is the web client id or the Android client id. It opens the partition from the body's `userId` and does not compare it to `sub`. It also requires the Functions `x-functions-key` header, which the Rule never forbids.
- M sends only `Authorization: Bearer` and no `userId`, expecting the API to take `sub`.

Nothing in AD-3 says the token is the only credential, that `sub` is read from the validated token, or that a client-supplied `userId` is rejected.

**Clash:** W and A stay up only when the body `sub` is whatever W felt like writing, so one account can read another (the isolation the spine never actually rules). M against that same A gets 400, missing `userId`, or 401, missing function key. First sign-in also forks: A inserts the account document and surfaces a loser of the race as 500; W treats that 500 as `Error reaching server` (AD-8) and stops, even though the account exists.

**Close it — tighten AD-3:** The only credential is `Authorization: Bearer <Google ID token>`. The API checks signature, expiry, and audience, and takes `userId` only from `sub`. A `userId` in the body, query, or route is a 400. No function key and no platform auth gate in front of that check. Creating the account document (`id` `account`) is an idempotent upsert; a concurrent first call that finds the document continues the request.

## 3. The read payload is not "the items"

**Location:** AD-1.

**Pair:** W and A.

**Letter:**

- A answers the partition read with every document, including the account document, because the Rule says not to filter, and with a continuation token after 100 items, because it still returns every item across pages. Each element is the stored document from finding 1.
- W treats the body as one JSON array of item wire objects and paints whatever comes back. It does not follow a continuation token (paging is under Deferred, so W has no token protocol to implement).

"Every item document" and "does not filter by state or by Today" still allow the account row, a wrapper object, and a page.

**Clash:** W either throws on the account row (no `description`) or shows one page and hides the rest of the history. Reload after save (AD-6, AD-8) then disagrees with what W just wrote.

**Close it — tighten AD-1:** One `GET /items` returns a single JSON array of the item wire objects in the caller's partition, done items included, account document excluded, no continuation token.

## 4. Create, replace, etag, batch, and delete are all unspecified

**Location:** AD-2, AD-4, AD-6, AD-8.

**Pair:** W and A.

**Letter:**

- **Create.** W quick-adds `{ description, dueDate }` and waits for a body that includes the new `id` and `etag`. A requires `state` and `position` on the body, assigns the GUID (AD-4), and returns 204 with an empty body. The new id exists only on the next read.
- **Update.** W sends a merge patch of the changed fields and the etag in `If-Match`, with the Cosmos quotes stripped. A ignores `If-Match`, reads `_etag` from the JSON including the quotes, and replaces the whole document. A missing `description` becomes empty and fails AD-7, or overwrites the stored description.
- **Reorder / multi-move.** Both send one request, so AD-2's "one request" and "transactional batch or nothing" hold. W posts `{ "items": [ ...full objects with etag... ] }`. A expects a bare array and one etag for the whole call.
- **Delete.** W sends DELETE plus the etag and drops the row. A sets `deleted: true` on the same document, returns 204, and leaves it in the partition. AD-7's state enum is untouched. AD-1 then returns the item again. AD-6's "deletes send the etag" is satisfied by both.

**Clash:** Create never yields an etag, so the next edit cannot satisfy AD-6. Patch against replace destroys fields. The batch body does not parse. Delete looks successful and the item returns on reload. W snaps back; A has kept a tombstone.

**Close it — tighten AD-2 and AD-6:**

- `POST /items` — client sends `description`, `dueDate`, and optional `detail`. The API sets `id`, `state` `not-started`, `completedDate` null, `position` to one more than the max `position` on that `dueDate` (0 when the date has no items), `userId` from `sub`, and `schemaVersion`. Response `201` is the wire object including `etag`. The API does not invent a due date from a clock.
- `PUT /items/{id}` — full wire object, not a merge patch.
- `POST /items/batch` — a JSON array of full wire objects. Used for reorder and for the web multi-move. One Cosmos transactional batch. Any etag miss writes nothing and returns 409.
- `DELETE /items/{id}` — physical delete of that document. `204`. No deleted flag and no leftover document.
- `etag` on update, batch, and delete is that document's Cosmos `_etag`, unchanged, quotes included, in the `etag` property. `schemaVersion` is not the etag (already said; keep it).

## 5. Two owners of order, and hide-completed writes two sequences

**Location:** AD-1, AD-4, AD-9, AD-10, consistency table "Positions are the integers the client submits for the affected due date."

**Pair:** W and M. Both persist through the same A, which stores whatever integers it is given and does not recompute them (AD-7: after the check, Core trusts the values; nothing tells A to own position).

**Letter:**

- W stores an integer `position` on each item and sorts due date, then position. The consistency table asks for integers submitted for the affected due date. W does that.
- M stores the order as an array of item ids on the account document. The account is a stored fact (AD-1, AD-4). The integers M submits are indexes into that array. Item documents have no `position`. Views are still computed on the device. This is not a stored Today list and not an items blob of the item documents themselves, so the Rules stand.
- A second pair, both using item `position`, still splits. Full order for a date: A (done, position 0), B (1), C (2). Hide completed is on, so the painted list is B, C (AD-9: the flag is device-local and changes no server state by itself). The user drags C above B.
  - W renumbers the painted rows only: C → 0, B → 1, A stays 0. With hide on, the user sees C, B.
  - M rebuilds the full date: remove C from [A, B, C], insert C in front of the visible neighbor B, write [A, C, B]. The user also sees C, B.
- Shared JSON examples do not yet exist. AD-10 requires them to state items, device date, and resulting order. It does not require `position` values, `hideCompleted`, or the operation. Both units can pass a fixture of not-done items with unique positions.

**Clash:** W never reads M's account array, so W's order is creation order (every `position` missing, tie-break unstable). M never reads W's `position`. On the hide-completed drag, unhide shows W's tie (A and C both at 0; `id` order picks the winner) and M's [A, C, B]. Same items, same device date, different order, which is the divergence AD-10 names and does not prevent. Two creates that both take "max + 1" also land on the same integer; without a published tie-break the clients can sort the tie either way and both match a fixture that has no ties.

**Close it — tighten AD-10, and make the consistency line a Rule:**

- `position` is a field on the item document only. The account document holds no order.
- Total order is `dueDate` ascending, then `position` ascending, then `id` ascending. State is not a sort key.
- For each due date the client writes positions across every item with that date, done items included. Hide-completed changes painting only. A drag rebuilds that full sequence: hidden items keep their relative order, and the dragged item is inserted beside the visible neighbor it was dropped against.
- A cross-day drag sets `dueDate` to the target day and inserts at that drop index in the target day's full sequence (finding 8 splits this from a button move).
- A move that has no drop index (tomorrow, next week, next month, multi-move) appends after the current max. A multi-move keeps the request order among items that share a target day.
- The API persists submitted positions. It assigns a position only on create (finding 4).
- `contracts/view-examples.json` rows carry the items (with `position` and `id`), `deviceDate`, `hideCompleted`, the operation, and the expected ids in order. xUnit and Vitest both load that schema.

## 6. Two clocks for the completed date

**Location:** AD-1, AD-7, AD-10.

**Pair:** W and A.

**Letter:**

- W, on the transition to `done`, sets `completedDate` to the device's local civil date and sends it. Leaving `done` sends `null`. Due date is unchanged (AD-10).
- A checks that a present completed date matches `YYYY-MM-DD` (AD-7), then overwrites it with `DateTime.UtcNow` formatted as a date whenever the new state is `done`, and clears it when the new state is not `done`. A is "recording the completed date". AD-1 forbids a server clock deciding Today. It does not mention this field. Core then trusts the value A wrote.

**Clash:** In the evening in the Americas, W records today and A stores tomorrow. History and any later "completed on this day" reading diverge after the reload AD-6 already requires. A client that sends `completedDate: ""` for "none" and an API that only accepts `null` or a date also 400 each other; AD-7 never says which encoding is "no date".

**Close it — one sentence on AD-10, plus the null rule in finding 1:** The client sets `completedDate` to the device's local civil date in the same request that sets `done`, and sets `null` in the request that leaves `done`. The API stores that value. It does not substitute a server clock. `completedDate` is `null` exactly when `state` is not `done`.

## 7. Today is undefined, and "device date" can mean UTC

**Location:** AD-1, AD-9, AD-10. The word "overdue" appears in AD-10 with no meaning.

**Pair:** W and M.

**Letter:**

- Both compute Today on the device from item facts and a date (AD-1, AD-9). Both recompute on foreground and run no midnight timer. Week starts Monday. Both would pass view examples that contain only not-done items due today.
- W's Today is: not-done items with `dueDate` before today, oldest date first, then every item due today, any state. A done item due last month is absent. "Overdue" is that first group. Device date is the local civil date (`getFullYear` / `getMonth` / `getDate`).
- M's Today is every item with `dueDate` on or before today, any state, hide-completed applied after. A done item from last month is in Today until the user hides completed. M's device date is `DateTime.UtcNow` formatted as `YYYY-MM-DD`, which is a date the device produced. Dragging an overdue item therefore includes done items, and AD-10 tells M to set their due date to today.

**Clash:** Around local midnight the two clients disagree on which calendar day "today" is, so the same partition paints two Todays. With the clocks aligned, a done item from the past is on M's Today and not on W's, and a drag on M rewrites that item's due date. AD-10's shared examples do not stop this while they omit a done item with a past due date.

**Close it — tighten AD-1 and AD-10:** Device date is the device's local civil calendar date as `YYYY-MM-DD`. Today is every not-done item with `dueDate` before that date (oldest first), then every item with `dueDate` equal to that date, any state. Overdue means the first of those groups. A done item with an earlier due date is not on Today. A week is every item whose `dueDate` falls in that Monday–Sunday. Hide-completed removes `done` from what is painted after membership is decided.

## 8. One gesture, two positions, both written in AD-10

**Location:** AD-10 Rule, the two sentences "A move to another date puts the item at the end of that date" and "A drag across a day reschedules to that day and lands at the drop position."

**Pair:** This one cannot be obeyed in full by two units, because one gesture is both a move to another date and a drag across a day. That is the hole. W and M each keep every other AD and one of the two sentences:

- W treats both sentences as live for a cross-day drag and applies them in written order. The drop index is applied, then "puts the item at the end" wins. Buttons do the same. Every cross-day gesture appends.
- M treats the drag sentence as the one that assigns a position, so a cross-day drag inserts at the drop index. "End of that date" is what M does when the drop index is the last row. A drop in the middle still complies with M's reading of "a position on that date."

**Clash:** Drop an overdue item between two items due today. W appends. M inserts in the middle. Both clients have implemented the Rule they were given. The JSON examples cannot rank them, because each example would have to violate one of the two sentences.

**Close it — replace those two sentences:** A drag that crosses a day sets the due date to that day and inserts at the drop index in that day's full sequence (finding 5), including a drag from overdue into today's items. A reschedule that has no drop index appends. Those are different gestures.

## 9. Field errors have no body

**Location:** AD-8.

**Pair:** W and A.

**Letter:** Both use ASP.NET Core problem-details. A 400 "carries field messages." W reads `errors.<wire field>[0]` (the framework's validation map, camelCase keys). A puts one string on `detail` and names the field in prose. Neither body contains an exception or a Cosmos message. 401, 409, and 5xx still follow the Rule.

**Clash:** W cannot attach the 400 to `description` or `dueDate`. The person sees a failed save with no field to correct, which is the client-invented error body AD-8 set out to stop.

**Close it — tighten AD-8:** `Content-Type: application/problem+json`. A 400 includes `errors`, a map from the wire property name to an array of strings. Clients highlight those properties. Other statuses stay as written.

## 10. Each environment exists, and neither client is told which one to call

**Location:** AD-2, AD-13.

**Pair:** W (local) and A (local), with M as the same kind of miss.

**Letter:**

- A local process hosts the Function App against the emulator. The free-tier account is untouched (AD-13).
- W's dev server calls the production Function App URL, which is still "the Function App URL" (AD-2). The production API is the only writer. No local process writes a production item; the production API does, on behalf of a dev session.
- M's debug build calls `https://localhost:7071`, which is the Function App on a machine the emulator cannot see. Release M calls production. Both are the Function App URL.

**Clash:** Dev W writes real items. Dev M cannot reach A's emulator, so sign-in and saves never meet the local API. AD-13's two environments stay intact and the clients still pick the wrong one.

**Close it — tighten AD-13:** Each client takes the Function App base URL from environment configuration. A local or debug build contains only the local Function host (on Android, the emulator's alias for the machine running that host). A release build contains only the production Function host.

## 11. Token names are "the same" without a list

**Location:** AD-12.

**Pair:** W and M.

**Letter:** W uses only CSS variables in `tokens.css` and ships one dark sheet selected for later by `data-theme`. M uses only resource-dictionary entries, dark only, no theme picker. W's names are `--bg`, `--surface`, `--text`, `--accent`, `--line`. M's names are `Background`, `Card`, `Foreground`, `Accent`, `Separator`. No component contains a hex value.

**Clash:** The names are not the same, so the dark sheet and the MAUI dictionary cannot be checked against one list. A later theme has two vocabularies. AD-12's Rule is satisfied on each side.

**Close it — tighten AD-12:** The token names are the DESIGN.md color keys, written the same in both places: `background`, `surface`, `line`, `text`, `text-muted`, `accent`, `surface-picked`, `error`. CSS custom properties use those names. The MAUI dictionary uses those names. Project 1 still ships dark only.

## Already closed

These are real ways to diverge that the Rules already forbid. They are not findings.

- A client writing Cosmos, or the phone calling the Static Web App for data (AD-2).
- A second identity provider, a shared session, or email as the user id (AD-3). The hole is which bytes supply `sub`, finding 2.
- One items document both clients overwrite, or dropping done items from storage (AD-4). The hole is an order array on the account, finding 5.
- Using `schemaVersion` as the concurrency token (AD-6).
- A third validator inside Core, or a different length or state enum (AD-7).
- A merge screen on 409, or exception text in the body (AD-6, AD-8).
- A server round trip or a midnight timer for hide-completed or Today (AD-9). Week-start Monday, next week = the following Monday, next month = the first, are already pinned.
- A storage interface, an infrastructure project, or the web project referencing Core (AD-11). That last ban is why finding 1 has to live in a JSON schema.
- A second Cosmos account, always-ready instances, or local API code aimed at the free-tier account (AD-13). Finding 10 is the client URL, which the Rule does not mention.
- Client idempotency keys and paging, which Deferred explicitly postpones. Compliant units tolerate duplicate creates on retry and return the whole partition in one response (finding 3 makes the "one response" explicit).

## What to change, and nothing else

| Hole | Action |
| --- | --- |
| 1. Wire item | Tighten AD-7; one sentence on AD-5 |
| 2. `userId` and first account write | Tighten AD-3 |
| 3. Read payload | Tighten AD-1 |
| 4. Writes and etag | Tighten AD-2 and AD-6 |
| 5. Owner of order | Tighten AD-10 |
| 6. Completed-date clock | Tighten AD-10 |
| 7. Today and civil date | Tighten AD-1 and AD-10 |
| 8. End versus drop index | Replace the two AD-10 sentences |
| 9. 400 `errors` map | Tighten AD-8 |
| 10. Client base URL | Tighten AD-13 |
| 11. Token names | Tighten AD-12 |

No new project, no new store, no change to the paradigm. The spine was not edited.
