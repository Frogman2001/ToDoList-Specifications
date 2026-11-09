# Adversarial review — architecture spine

**Verdict: Fail.** Two units one level down can obey every adopted Rule and still not compose. The breaks that remain are missing wires from the function host module to Cosmos, two legal shapes for the same JSON, and two legal writers of `position`.

Target: `ARCHITECTURE-SPINE.md` (final, updated 2026-11-06) and companion `1-4_OutputDocs/terraform.md`. Spine not modified. Terraform guide not modified.

## Units

Built from the Rules and, for the Terraform pair, from the sentences in `terraform.md` that say who owns which resource and which values the root passes. Each unit treats `Prevents` as motive and the Rule as the constraint. Diagrams are not Rules.

- **T-root** — root stack. Production resource group, Cosmos account, database `todolist`, container `documents`, Static Web App, module call. Remote state as written.
- **T-mod** — `modules/function_host` only. Plan, Function App, user-assigned identity, function storage account, container `app-package`, four storage data roles, Cosmos SQL role assignment.
- **A** — `ToDoList.Api` plus the concrete Cosmos class. Only writer. Reads settings by the AD-14 names.
- **C** — `web`, and `ToDoList.App` where the phone is the other party. Computes views. Does not reference Core or Scheduling from `web`.

## 1. The module never receives the Cosmos endpoint or the Google audiences

**Location:** AD-14 app-settings list, AD-15 module boundary, `terraform.md` "The root passes the Cosmos account id and the Static Web App hostname into the module" and the outputs row "Cosmos endpoint | already written to `COSMOS_ACCOUNT_ENDPOINT`".

**Pair:** T-root and T-mod. A consumes the result.

**Letter:**

- T-root creates the Cosmos account and the two Google client id variables in `prod.tfvars`. It passes the Cosmos account id and the Static Web App hostname into the module, and nothing else, because that is the pass list. It outputs the account's document endpoint for humans. It does not own the Function App, so it does not write app settings.
- T-mod creates the Function App and sets every AD-14 name. The only Cosmos value it was given is the account id, so `COSMOS_ACCOUNT_ENDPOINT` is that id. `GOOGLE_WEB_CLIENT_ID` and `GOOGLE_ANDROID_CLIENT_ID` are set to empty strings because those variables never arrive. The Cosmos SQL role assignment is scoped to the account id with definition `00000000-0000-0000-0000-000000000002`. `location` on the plan defaults inside the module; the root's `location` variable is a different variable that happens to share a name.

Both files match the ownership table. The role assignment is on the real account. The setting the SDK will open is not an endpoint.

**Clash:** A builds `CosmosClient` from `COSMOS_ACCOUNT_ENDPOINT` and the call fails before any item is written. The identity could have written, and the role will not help. Sign-in fails closed for a second reason: the token audience is compared to an empty setting, so every bearer is rejected. A root applied in one region and a module left on its default region also split the "one region" rule without either file using two regions itself.

**Close it — tighten AD-15:** The module's inputs are the Cosmos account id, the Cosmos document endpoint (the account's endpoint attribute, not a URL built from the name), `location`, `name_suffix`, the Static Web App hostname, and the two Google client ids. The module writes `COSMOS_ACCOUNT_ENDPOINT` from that endpoint and the two Google settings from those ids. `COSMOS_DATABASE_NAME` is `todolist` and `COSMOS_CONTAINER_NAME` is `documents`. One `location` value is passed through; the module has no location default.

## 2. Session consistency plus a host that forgets the session

**Location:** AD-14 "Cosmos is Session consistency" and "Always-ready instances stay zero"; `terraform.md` "Leave the scale-out cap at the platform default"; AD-1 the reload reads Cosmos; AD-6 the client reloads after 409.

**Pair:** A and C, against the host T-mod built.

**Letter:**

- T-mod sets Session consistency, always-ready count 0, and does not set a maximum instance count. No virtual network. That is the Rule.
- A uses the account endpoint and the managed identity, keeps no item cache, and does not put a Cosmos session token on any response. AD-14 does not name one. Each instance builds its own client.
- C, after 201 or after a 409, calls `GET /items` and trusts that body as the partition.

**Clash:** The instance that wrote the item can be gone on the next request (scale to zero, or a second instance under the default cap). Session consistency gives read-your-writes only inside the client session that holds the token. A new process does not hold it. The 201 C just applied is missing from the reload, so the item vanishes until some later read. C followed the reload Rule and A followed the consistency Rule.

**Close it — tighten AD-14:** Keep Session. The API echoes the Cosmos session token on every response as the header `x-ms-session-token` and accepts that header on the next request, passing it into the SDK. No item cache. Do not pin the scale cap to 1 and call that a fix: a cold start is a new session either way.

## 3. The host gate in front of the bearer token

**Location:** AD-2 route list, AD-3 "A function key is not identity", AD-13 `apiBaseUrl` is the Function App host, AD-14 settings list (no key).

**Pair:** A and C. T-mod is silent, which both treat as permission.

**Letter:**

- A registers the five routes under the names AD-2 gives (`GET /items`, and the rest). It leaves the Functions HTTP prefix at the platform default `api`, because that prefix is the host, not the route name. It sets the trigger auth level to `function`. The key is not the user id; `userId` still comes only from `sub`, and a body `userId` is ignored. The Google check still runs after the platform check.
- C sets `apiBaseUrl` to `https://` plus the hostname Terraform outputs. It calls `{apiBaseUrl}/items` with `Authorization: Bearer` only. No AD lists a function key, so C does not send one.

**Clash:** The platform answers 401 before the Google check. C treats 401 as "start sign-in" (AD-8) and loops, because a new Google token does not add `x-functions-key`. A second cut of A that uses auth level anonymous still 404s, because C calls `/items` and the function is mounted at `/api/items`.

**Close it — tighten AD-2 and AD-3:** `host.json` route prefix is empty. HTTP triggers are anonymous. The only accept test is the Google bearer check in AD-3. `apiBaseUrl` is `https://` plus the Function App default hostname, with no path. Terraform's hostname output is that hostname and nothing more.

## 4. GET has no envelope, and the writes that must be retried have no body

**Location:** AD-1 `GET /items` returns every item; AD-2 success body is specified for `POST /items` only; AD-6 etag on the next update; `POST /items/changes` request shape `{ "items": [ ... ] }`.

**Pair:** A and C.

**Letter:**

- A returns the partition's item documents as `{ "items": [ ... ] }`, matching the only collection shape the Rules name. `POST /items` returns 201 and the stored item. `PUT`, `DELETE`, and `POST /items/changes` return 204 with an empty body. The writes happened. Etag mismatches still return 409 and write nothing.
- C reads `GET /items` as a JSON array, because AD-1 says it returns the items and does not name a wrapper. After a successful batch, C keeps the etags it sent and paints the new order. AD-6 tells C to reload on 409, not on 204.

**Clash:** C throws on the wrapped GET, or A throws on the bare array, depending on which sentence each side generalized. When the envelope happens to match, the next edit still 409s: Cosmos moved `_etag` and C still holds the pre-batch value, so every reorder costs a failed write and a full reload. A batch that failed for a vanished id has no status either (finding 8), so C cannot tell "reload" from "Error reaching server".

**Close it — tighten AD-1 and AD-2:**

- `GET /items` is one JSON array of wire items. No wrapper, no continuation token, account document not included.
- `PUT /items/{id}` returns 200 and the wire item with the new etag.
- `DELETE /items/{id}` returns 204.
- `POST /items/changes` returns 200 and `{ "items": [ ... ] }` in request order, each with the new etag. The client replaces its copies from that body.

## 5. The etag bytes are not specified

**Location:** AD-6 "copied from the Cosmos `_etag`" and "that value in the `If-Match` header"; AD-7 `etag` is a JSON field.

**Pair:** A and C.

**Letter:**

- A copies the SDK value into JSON unchanged. That value includes the quote characters Cosmos puts on `_etag`. On write, A compares the raw `If-Match` header to that SDK value.
- C stores the JSON string and sends it as a normal quoted HTTP header. The header parser strips one layer of quotes. C also strips a layer when reading JSON, because a JSON string of an etag should not itself contain quote characters.

Both copied `_etag`. Both used `If-Match`. Neither invented a different concurrency token (`schemaVersion` stays the schema field).

**Clash:** Every `PUT`, `DELETE`, and batch 409s. C snaps back and reloads, gets the same bytes, and 409s again. The item cannot be edited.

**Close it — tighten AD-6:** The JSON `etag` is the Cosmos `_etag` with the surrounding quote characters removed. `If-Match` carries that same sequence as an HTTP quoted-string. The API compares the unquoted values. Missing or mismatched still writes nothing.

## 6. Create's body and PUT's omitted fields

**Location:** AD-2 "`PUT` replaces the editable fields and keeps `id`, `userId`, and `schemaVersion`"; AD-7 required-ness is stated for description only; AD-10 a new item is placed at the end, and setting done stores the client `completedDate`.

**Pair:** A and C.

**Letter:**

- C quick-adds `{ "description", "dueDate" }`. Description is the only field AD-7 marks required. On a later rename, C sends `PUT` with `description` and `If-Match` only, which replaces the editable field it changed and leaves the server-owned three alone.
- A requires `state` and `position` on create, because AD-7 says state is one of the three values and AD-10 says position is a unique integer, so a missing one fails the check. On `PUT`, A replaces the document: an omitted `detail`, `state`, `dueDate`, `completedDate`, or `position` is written null, then the state enum check 400s.

A second A defaults create to `state` `not-started`, `completedDate` null, and `position` max+1, and merges `PUT`. That A also fits the sentences. C cannot tell which A it is calling.

**Clash:** Against the strict A, quick-add and rename never succeed (400), so the 201-with-etag Rule is unreachable from the client the field list actually allows. Against the merge A, a client that thought "replace" and omitted `detail` keeps the old detail, while a replace A would have cleared it. Completed-date and position follow whichever accident the pair picked.

**Close it — tighten AD-2 and AD-7:**

- `POST /items` body is `description` (required), `dueDate` (`YYYY-MM-DD` or null), and optional `detail`. The API sets `state` to `not-started`, `completedDate` to null, and `position` to one more than the max on that due date (0 when none). Response 201 is the wire item.
- `PUT /items/{id}` body is the full wire item except that `id` is in the route. `description`, `detail`, `state`, `dueDate`, `completedDate`, and `position` are all present. An absent field is 400. The API does not merge, and it does not null a field the client left out.
- `detail` omitted on create is null. Empty string is 400. `completedDate` is null exactly when `state` is not `done`.

## 7. Two owners of `position`

**Location:** AD-2 the batch "writes those items"; AD-7 after the check, Core trusts the values; AD-9 hide-completed does not change `position`; AD-10 "`position` is a unique integer per due date, owned by the item", "placed at the end", "that request renumbers the affected date".

**Pair:** C and A. A second pair is web C and Android C, with A persisting whatever integers it is given.

**Letter:**

- C-web, on a drag, sends only the moved item. `position` is the drop index. "That request renumbers" is the API's job, and AD-2 says the API is the only writer. Hide-completed is on, so the hidden done items are not in the gesture (AD-9: the flag does not change their positions).
- A writes exactly the objects in `{ "items": [...] }` and does not load the rest of the date. AD-2 says the batch writes those items or none. It checks that each `position` is an integer, then trusts it. It does not recompute uniqueness; AD-10's sort already breaks ties with `id`.
- C-android, same A, renumbers every item on that due date, hidden done items included, and sends the full set. "Owned by the item" means the client that knows the finger position writes every integer. This also satisfies the Rules, and it does not match C-web.

**Clash:** C-web against this A stores the drop index on one document and leaves the siblings. Two items share an integer. Sort uses `id`, so the row does not land where it was dropped. C-android against the same A lands correctly. After unhide, the two clients disagree on the full date. Shared examples that omit `hideCompleted` and a collision pass both. Create has the same fork: C sends max+1, a second tab sends the same max+1, A stores both, and `id` order quietly picks a winner the examples never assert.

**Close it — tighten AD-10:**

- The client owns the integers. The API persists them. The API assigns `position` only on create, as in finding 6.
- A drag sends every item on each affected due date, done items included, with positions rewritten to `0..n-1` in the new order. Hidden items keep their relative order. The dropped item is inserted beside the visible neighbor it was dropped on.
- A reschedule with no drop index appends after the current max on the target date.
- `contracts/view-examples.json` rows include `position`, `id`, `deviceDate`, `hideCompleted`, the operation, and the expected ids in order. xUnit and Vitest both run that file.

## 8. An unknown `schemaVersion` fails "the read"

**Location:** AD-5 "A `schemaVersion` the code does not know fails the read." AD-8 5xx shows `Error reaching server`. No status for a single bad document, a missing id, or a batch that is not an etag miss.

**Pair:** two cuts of A, both composed with the same C.

**Letter:**

- A-strict treats the partition read as one read. One stored `schemaVersion` it does not know fails that read. It returns 500 problem-details with no Cosmos text.
- A-skip fails that document's conversion, omits it, and returns 200 for the rest. The unknown version did fail its read.
- Neither cut has a status for `PUT`/`DELETE` of an id that is not in the partition, or for a batch Cosmos rejects for a reason other than etag. One uses 404, the other 500. Etag misses stay 409.

**Clash:** The same partition is "Error reaching server" on one API and a quietly shorter list on the other. C cannot tell a skipped item from a deleted one. A 404 on a stale id does not snap the row back (that path is 409 only), and a 500 does not either unless C treats every failure as snap-back, which AD-8 does not say.

**Close it — tighten AD-5 and AD-8:** An unknown `schemaVersion` fails the whole call with 500. The client shows `Error reaching server` and does not drop the local row. `PUT` or `DELETE` of an id that is not in the caller's partition returns 404 and writes nothing. A batch that fails for any reason other than a missing or mismatched etag returns 409 when an etag mismatches and 400 otherwise, and writes nothing. 404 snaps back and reloads, same as 409.

## 9. The 400 `errors` map has no member type

**Location:** AD-8 "400 uses an `errors` map of field name to messages."

**Pair:** A and C.

**Letter:**

- A returns `Content-Type: application/problem+json` and `errors` as a map of field name to one string. No exception text.
- C reads `errors.description` as an array of strings, because "messages" is plural and that is the ASP.NET validation shape. It shows `errors.description[0]`.

**Clash:** The 400 body parses and the field message does not. The form cannot point at `description` or `dueDate`, which is the client-invented error body AD-8 exists to prevent. A cut of A that uses a string array clashes with a C that reads a string. Both maps satisfy the sentence.

**Close it — tighten AD-8:** 400 `errors` is a JSON object whose values are arrays of strings, keyed by the wire field name. Clients show those strings on those fields.

## 10. The production Cosmos client and the emulator client are not the same construction

**Location:** AD-14 settings and "On a developer machine the API uses the emulator connection string from local settings and does not use this identity"; AD-13 Terraform does not describe the local machine; stack row for the emulator.

**Pair:** A-prod and A-local, two construction paths inside the one API, plus the Flex host from T-mod.

**Letter:**

- A-prod constructs `CosmosClient` with the endpoint and `new ManagedIdentityCredential()` with no client id argument. AD-14 lists `AZURE_CLIENT_ID` but does not say the SDK is `DefaultAzureCredential` or that this constructor must be given that id. Connection mode stays the SDK default, Direct.
- A-local looks for the connection string under `ConnectionStrings:Cosmos`. A machine set up from the prose looks for `COSMOS_CONNECTION_STRING`. Both are "the emulator connection string from local settings". The emulator has no Entra login, so the identity path is correctly skipped.
- T-mod does not set a Cosmos key and does not set a connection mode. Direct mode is not a key.

**Clash:** Production calls fail closed when the credential binds to the wrong identity or to none, even though `AZURE_CLIENT_ID` is present on the app. Direct mode opens a connection per partition range; the Flex host then fails Cosmos calls that Gateway would have served. Local runs only for the author whose setting name matches. The other author, still inside the Rules, cannot sign in against the emulator.

**Close it — tighten AD-14:** Production is `CosmosClient(COSMOS_ACCOUNT_ENDPOINT, new DefaultAzureCredential())` with Gateway connection mode. `AZURE_CLIENT_ID` and `AzureWebJobsStorage__clientId` are both the user-assigned identity's client id. Local settings use the name `COSMOS_CONNECTION_STRING` and that path only; production does not set it. Gateway mode is also what the vNext emulator speaks.

## Already closed

These are real ways to diverge that the current Rules already forbid. They are not findings. The 2026-11-04 adversary review (`reviews/review-adversary.md`) raised several of them against the draft; the adopted text now stops them.

- CamelCase wire fields, the field list, and server-owned `id` / `userId` / `schemaVersion` (AD-7, AD-5).
- `userId` taken only from token `sub`, body `userId` ignored, email not the id (AD-3).
- Account document id `account`, no item array, one document per item, database `todolist`, container `documents`, partition `/userId` (AD-1, AD-4, `terraform.md`).
- Physical `DELETE`, batch body `{ "items": [ ... ] }`, all-or-nothing batch, 201 with `id` and `etag` on create (AD-2, AD-6).
- Device date is the local calendar date, not UTC. Today membership, overdue, week, drag-versus-reschedule, and completed date from the client are specified (AD-1, AD-10).
- Token names are listed (AD-12). 401 / 409 / 5xx client behavior is specified (AD-8). The 400 member type is finding 9, not the status table.
- `apiBaseUrl` exists per environment (AD-13). Finding 3 is the path and the platform gate, which that Rule still does not name.
- CORS inputs that are already wired: the root passes the Static Web App hostname, and the allow-list adds `http://localhost:5173` (`terraform.md`). Storage public network stays on, blob public access stays off, so the Flex host can reach `app-package` without a private endpoint.
- Cosmos SQL role id `00000000-0000-0000-0000-000000000002` at account scope, inside the module, not a control-plane role (`terraform.md`, AD-15).
- A second Terraform module, state in git, or Terraform publishing the zip (AD-15).

## What to change, and nothing else

| Hole | Action |
| --- | --- |
| 1. Module inputs | Tighten AD-15 with the pass list |
| 2. Session token | Tighten AD-14 |
| 3. Anonymous routes at the host root | Tighten AD-2 and AD-3 |
| 4. GET array and write responses | Tighten AD-1 and AD-2 |
| 5. Etag characters | Tighten AD-6 |
| 6. Create body and full PUT | Tighten AD-2 and AD-7 |
| 7. Owner of position | Tighten AD-10 |
| 8. Failed read and unknown id | Tighten AD-5 and AD-8 |
| 9. 400 `errors` values | Tighten AD-8 |
| 10. Cosmos client construction | Tighten AD-14 |

No new project, no new store, no change to the paradigm. The spine and `terraform.md` were not edited.
