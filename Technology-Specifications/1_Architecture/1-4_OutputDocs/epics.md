---
status: final
updated: 2026-11-08
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-ToDoList-2026-10-30/prd.md
  - _bmad-output/planning-artifacts/prds/prd-ToDoList-2026-10-30/addendum.md
  - _bmad-output/planning-artifacts/ux-designs/ux-ToDoList-2026-10-30/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-ToDoList-2026-10-30/EXPERIENCE.md
  - Technology-Specifications/1_Architecture/1-4_OutputDocs/architecture.md
  - Technology-Specifications/1_Architecture/1-4_OutputDocs/terraform.md
  - Technology-Specifications/1_Architecture/1-3_Working/architecture-ToDoList-2026-11-04/ARCHITECTURE-SPINE.md
  - Technology-Specifications/1_Architecture/1-3_Working/architecture-ToDoList-projects-2026-11-06/ARCHITECTURE-SPINE.md
---

# ToDoList - Epic Breakdown

## Overview

This is the architect's handoff for ToDoList project 1. It breaks the work into three areas that can be finished apart: the backend, the web app, and the Android app.

The backend is finished when a caller can use the HTTP API against the database, with no screen in front of it. The web epic and the Android epic each call that API. Neither client epic depends on the other. Shared rules stay in one place: `contracts/item-fields.json` is introduced by the backend, and `contracts/view-examples.json` is the one view fixture both clients run. Whichever client is built first adds the cases it needs. The other client runs that same file and does not keep a second copy.

The product owner keeps the product script. The PRD and `EXPERIENCE.md` stay the source for wording, journeys, and interaction. A client story states the architectural outcome and points at the backend story for the HTTP contract.

The developer uses this as input, then reads the PRD and the architecture again and decides the work. A story here may be split. Detail may be added.

Each story has a Notes section. Notes say where the work sits, which tools to use, and a size sketch taken from the project-structure draft. Notes are not acceptance criteria. The size sketch is not a schedule. The project-structure draft puts the whole application at about 60 authored files, with the time in phone gestures, web drag and keyboard, the date field, and the Cosmos etag and batch path.

No starter template is specified. Each story adds the documents, routes, and projects that story needs. The publish pipeline that ships the Function App zip and the Static Web App files is out of this handoff, matching "Left for later" in `architecture.md`. The backend can still be called on a developer machine before that pipeline exists.

## Requirements Inventory

### Functional Requirements

- FR-1: Sign in with Google, and create the user on first sign-in. PRD §4.1.
- FR-2: Sign out, and show no items afterward. PRD §4.1.
- FR-3: Add an item with a description only. PRD §4.2.
- FR-4: Today view from the device date. PRD §4.3.
- FR-5: Specific-date view, including done items. PRD §4.3.
- FR-6: Week view, Monday to Sunday. PRD §4.3.
- FR-7: Hide completed, and show them again. PRD §4.3.
- FR-8: Edit description, detail, and due date. PRD §4.4.
- FR-9: Change state. Done records the completed date and does not change the due date. Leaving done clears the completed date. PRD §4.4.
- FR-10: Delete after confirmation. PRD §4.4.
- FR-11: Reschedule by a picked date, tomorrow, next week, or next month. PRD §4.5.
- FR-12: Reorder by dragging, including onto another day. PRD §4.6.
- FR-13: The same behavior on web and Android, except FR-15. PRD §4.7.
- FR-14: Web opens on the week that contains today. Android opens on Add new. PRD §4.7.
- FR-15: Move several items at once, web only. PRD §4.7.

### NonFunctional Requirements

- NFR-1: A person cannot see or change another person's items. PRD §4.8.
- NFR-2: No item data without signing in. PRD §4.8.
- NFR-3: The same items and the same date produce the same views and order on both clients. PRD §4.8.

### Additional Requirements

- AD-1, AD-10: Cosmos stores account and item facts. Clients compute Today, week, and order. `GET /items` returns every item in the caller's partition and does not filter.
- AD-2: Both clients call the Function App. Routes are `GET /items`, `POST /items`, `PUT /items/{id}`, `DELETE /items/{id}`, and `POST /items/changes`. No `/api` prefix. No function key.
- AD-3: Each client has its own Google ID token. `userId` is the token `sub`. Settings are `GOOGLE_WEB_CLIENT_ID` and `GOOGLE_ANDROID_CLIENT_ID`.
- AD-4, AD-5: Database `todolist`, container `documents`, partition `userId`. Account id `account`. Item id is an API-assigned GUID. `schemaVersion` starts at 1. Storage types are `AccountDocumentV1` and `ItemDocumentV1`.
- AD-6: `etag` is the Cosmos `_etag`. Updates, deletes, and batches require it. A mismatch is 409 and writes nothing.
- AD-7: `contracts/item-fields.json` is the only field definition. The client checks, the API checks once, then Core trusts the values.
- AD-8: Problem-details. 400 names the field. 401 starts sign-in. 409 snaps back and reloads. 5xx or no response shows `Error reaching server`.
- AD-9: Hide completed and the device date stay on the device. Today refreshes when the app returns to the foreground.
- AD-11, AD-16, AD-17, AD-21: Projects are `ToDoList.Api`, `ToDoList.Core`, `ToDoList.Scheduling`, `ToDoList.App`, `ToDoList.Tests`, and `web`. No storage interface. The API does not reference Scheduling. Tests for rules are xUnit and Vitest, not an API test pyramid.
- AD-12, AD-19, AD-20: Dark tokens only. One session type per client. No component kit, drag library, date-picker library, Redux, or React Router.
- AD-13, AD-14, AD-15: One production resource group, described in `terraform.md`. Local work uses the Cosmos emulator. Terraform does not publish the app.
- Deferred, and not given a story here: iOS, other sign-in providers, sharing, offline, admin tools, paging, a per-user week start, a phone theme picker, a background migration, idempotency keys, and the publish pipeline. PRD §5 and §6.2, architecture "Left for later".

### UX Design Requirements

- UX-1: Web Sign In before the list. Android opens on Add new and asks to sign in only when a save needs it. `EXPERIENCE.md` Foundation, Information Architecture, State Patterns.
- UX-2: The words in Voice and Tone, including `Error reaching server`, `Nothing here.`, and `Delete this item?`.
- UX-3: One day box. Add popup. Date field accepts a calendar or typed digits. State box cycles not-started, in-progress, and done. `EXPERIENCE.md` Component Patterns.
- UX-4: Hide completed is a top-bar action in any view.
- UX-5: Loading, empty, error, failed add, and failed move. `EXPERIENCE.md` State Patterns.
- UX-6: Phone gestures and web drag, including drop across a day. `EXPERIENCE.md` Interaction Primitives.
- UX-7: Web keyboard shortcuts in that same section.
- UX-8: Accessibility floor in `EXPERIENCE.md`.
- UX-9: Column counts in Responsive & Platform.
- UX-10: Dark theme. No color hard-coded in a component. `DESIGN.md` and AD-12.
- UX-11: No stored lists, no labels, words rather than mystery icons. Inspiration & Anti-patterns.

### FR Coverage Map

- FR-1: Epic 1 creates the account on the first authenticated call. Epic 2 and Epic 3 each sign in with their own Google client.
- FR-2: Epic 2 and Epic 3 — sign out discards that client's token only.
- FR-3: Epic 1 stores the item. Epic 2 and Epic 3 each add through `POST /items`.
- FR-4: Epic 2 and Epic 3 — each client computes Today. Epic 1 returns every item and does not filter.
- FR-5: Epic 2 and Epic 3 — one date, computed on the client.
- FR-6: Epic 2 and Epic 3 — the week, computed on the client.
- FR-7: Epic 2 and Epic 3 — hide completed stays on the device.
- FR-8: Epic 1 replaces the item. Epic 2 and Epic 3 each edit.
- FR-9: Epic 1 stores `completedDate` as sent. Epic 2 and Epic 3 each change state.
- FR-10: Epic 1 deletes. Epic 2 and Epic 3 each confirm, then delete.
- FR-11: Epic 2 and Epic 3 — each client chooses the new date and position, then `PUT`.
- FR-12: Epic 1 writes the batch. Epic 2 drags with pointer events. Epic 3 drags with the phone gesture.
- FR-13: Epic 2 plus Epic 3. Neither epic is a substitute for the other.
- FR-14: Epic 2 opens on the week that contains today. Epic 3 opens on Add new.
- FR-15: Epic 1 writes the batch. Epic 2 moves several items. Epic 3 has no multi-select.
- NFR-1: Epic 1 — the partition is the token `sub`.
- NFR-2: Epic 1 — no data route without a bearer token. Epic 2 and Epic 3 do not call data routes while signed out.
- NFR-3: Epic 2 and Epic 3 — both run `contracts/view-examples.json`.
- UX-1: Stories 2.1 and 3.1
- UX-2: Stories 2.2, 2.5, 3.2, 3.5
- UX-3: Stories 2.2, 2.3, 2.4, 2.9, 3.2, 3.3, 3.4, 3.9
- UX-4: Stories 2.7 and 3.7
- UX-5: Stories 2.2, 2.3, 2.6, 3.2, 3.3, 3.6
- UX-6: Stories 2.11 and 3.11
- UX-7: Story 2.13
- UX-8: Story 2.13, and both clients keep the floor
- UX-9: Stories 2.9 and 3.9
- UX-10: Stories 2.1 and 3.1
- UX-11: Stories 2.1, 2.6, 3.1, 3.6
- AD-13, AD-14, AD-15: Stories 1.6 and 1.7
- Publish pipeline and the other deferred items: no story

## Epic List

### Epic 1: Backend
A caller with a Google token can read and write only that caller's items. The database and the HTTP API are enough. No screen is required.
**FRs covered:** the stored-fact half of FR-1, FR-3, FR-8, FR-9, FR-10, FR-12, and FR-15. NFR-1, NFR-2. AD-13, AD-14, AD-15.

### Epic 2: Web
A person can sign in, keep items, and arrange the day in the browser. This epic calls Epic 1. It does not require the Android app.
**FRs covered:** FR-1, FR-2, FR-3, FR-4, FR-5, FR-6, FR-7, FR-8, FR-9, FR-10, FR-11, FR-12, FR-14, FR-15, and the web half of FR-13. NFR-3.

### Epic 3: Android
A person can sign in, keep items, and arrange the day on the phone. This epic calls Epic 1. It does not require the web app.
**FRs covered:** FR-1, FR-2, FR-3, FR-4, FR-5, FR-6, FR-7, FR-8, FR-9, FR-10, FR-11, FR-12, FR-14, and the Android half of FR-13. NFR-3. No FR-15.

## Epic 1: Backend

A caller proves this epic by sending HTTP requests. The first stories run against the Cosmos emulator. The last two add the one production stack and keep the emulator off that stack. Clients compute Today, week, and order. This epic stores the facts they send and returns them.

### Story 1.1: Read my items

As a caller with a Google token,
I want to read every item stored for me,
So that a client can load my list without a screen on the server.

**Acceptance Criteria:**

**Given** a bearer token whose audience is `GOOGLE_WEB_CLIENT_ID` or `GOOGLE_ANDROID_CLIENT_ID`
**When** the caller sends `GET /items`
**Then** the API creates the account document with id `account` and type `account` when that partition does not already have one
**And** `userId` is the token `sub`, and a body `userId` is ignored
**And** a race that finds the account document already there returns it (AD-3, AD-4)
**And** the response is a JSON array of every item document in that partition, each including `etag` copied from the Cosmos `_etag`
**And** the array is not filtered by state, Today, or date, and the Function App does not cache it (AD-1)
**And** the server sets account `schemaVersion` to 1 and ignores a client value (AD-5)

**Given** a missing token, a rejected token, or a token for another audience
**When** `GET /items` is called
**Then** the response is 401 problem-details and nothing is written (AD-8, NFR-2)

**Given** items stored for one `sub`
**When** a different `sub` calls `GET /items`
**Then** those items are absent (NFR-1)

**And** this story adds `ToDoList.Api` auth, the account types in `ToDoList.Core`, and the concrete Cosmos class. There is no storage interface and no account route (AD-2, AD-16, AD-21).
**And** on a developer machine the Cosmos class uses the emulator connection string from local settings, against database `todolist` and container `documents` (AD-13).

**Notes:**

Where: this story creates the application host. `ToDoList.Api` gets `Program.cs`, `host.json`, `local.settings.json.example`, `Auth/GoogleToken.cs`, and `Data/CosmosDocuments.cs` for the partition read and the account document. `ToDoList.Core` gets `Account`, `AccountDocumentV1`, and the start of `DocumentConverter`. Solution glue is `ToDoList.sln`, `.gitignore`, and a short README. Item routes are not in this story. Later stories add them to `Presentation/ItemsEndpoints.cs` and the same Cosmos class.

Tools: .NET 10 isolated worker with the ASP.NET Core integration, `Google.Apis.Auth` 1.78.0, `Microsoft.Azure.Cosmos` 3.63.3, and the Cosmos vNext emulator in Docker (`mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest`). Prove it by calling `GET /items`. There is no API test project. xUnit does not reference the API.

Size: large for the backend. The project-structure draft puts the whole API at about 7 authored files, and this story is most of that skeleton plus the account half of Core. Later routes are additions, not a new host.

**Covers:** FR-1, NFR-1, NFR-2.

**Open for the developer:** how the emulator database and container are created locally. The names are fixed.

### Story 1.2: Add an item

As a caller,
I want to store one new item,
So that a description and the fields I send become a document in my partition.

**Acceptance Criteria:**

**Given** a valid bearer token and a body that passes `contracts/item-fields.json`
**When** the caller sends `POST /items`
**Then** the API checks the contract once and Core does not check again after that (AD-7)
**And** the API assigns the item id, sets `userId` from the token and `schemaVersion` to 1, and ignores client values for those three fields (AD-4, AD-5)
**And** the response is 201 with the stored item, including `id` and `etag`
**And** the document type is `item`, and `position`, `dueDate`, `state`, `detail`, and `completedDate` are stored as sent
**And** the API does not assign `position`, does not compute Today, and does not retry the create (AD-2, AD-10, AD-17)

**Given** a body that fails the field contract
**When** the caller sends `POST /items`
**Then** nothing is written, and the response is 400 problem-details with an `errors` map of field name to messages (AD-7, AD-8)

**And** this story adds `POST /items`, `ItemDocumentV1`, and `contracts/item-fields.json` as the only edited copy of the limits (AD-18).
**And** description is required, trimmed, 1 to 200 characters. Detail is optional, up to 2000. State is `not-started`, `in-progress`, or `done`. Dates are `YYYY-MM-DD` or null.

**Notes:**

Where: `Presentation/ItemsEndpoints.cs` gains `POST /items`. `CosmosDocuments` gains create. `ToDoList.Core` gains `Item`, `ItemDocumentV1`, and `ItemRules`. `contracts/item-fields.json` is created here as the only edited copy of the limits. `ToDoList.Tests` can start with `ItemRulesTests` and `DocumentConverterTests`. The API calls `ItemRules` once, then the converter and Cosmos trust the values. No Scheduling project. The API does not choose `position`.

Tools: System.Text.Json, camelCase, matching the contract file. xUnit 3 on `net10.0` for the rules and the converter. The same HTTP client as Story 1.1 for the route.

Size: medium, once Story 1.1 is running. Core's whole design is about 7 files, and the field check plus the converter are that design. This story is that work, not a new service.

**Covers:** FR-3.

### Story 1.3: Replace an item

As a caller holding an item's etag,
I want to replace that item's editable fields,
So that a stale copy cannot overwrite a newer save.

**Acceptance Criteria:**

**Given** an item id and the current `etag`, including any quotes, in `If-Match`
**When** the caller sends `PUT /items/` plus that id
**Then** the API replaces the editable fields and keeps `id`, `userId`, and `schemaVersion` (AD-2, AD-6)
**And** a client-sent `schemaVersion` is ignored (AD-5)
**And** `completedDate` is stored as sent. The API does not replace it with the server clock (AD-10)

**Given** a missing `etag`, or an `etag` that is not the current Cosmos `_etag`
**When** the PUT is sent
**Then** the API writes nothing. A mismatch returns 409 (AD-6)

**Given** a stored `schemaVersion` this code does not know
**When** `GET /items` reads that document
**Then** the read fails (AD-5)

**And** this story adds PUT. It does not add the batch route.

**Notes:**

Where: `PUT` is another action on `ItemsEndpoints` and `CosmosDocuments`. `If-Match` is the Cosmos `_etag`. The unknown-`schemaVersion` failure is the read path in `DocumentConverter`, exercised by the existing `GET /items`. No new project and no new package.

Tools: the same Function App and emulator. An HTTP client sends `If-Match`, including the quotes. xUnit covers a known version round-trip and an unknown version failing the read.

Size: medium. The project-structure draft calls the etag mismatch path one of the two parts of the API that take time. The surface is still the same two types.

**Covers:** FR-8, FR-9.

### Story 1.4: Delete an item

As a caller holding an item's etag,
I want to delete that item,
So that a stale caller cannot delete a newer copy.

**Acceptance Criteria:**

**Given** the current `etag` in `If-Match`
**When** the caller sends `DELETE /items/` plus the item id
**Then** that document is removed (AD-2, AD-6)

**Given** a missing or mismatched `etag`
**When** the DELETE is sent
**Then** nothing is written. A mismatch returns 409 (AD-6)

**Notes:**

Where: `DELETE` is another action on `ItemsEndpoints` and `CosmosDocuments`, using the same etag rule as replace. No new type.

Tools: the same HTTP client and emulator.

Size: small. It is the delete branch of the conflict path Story 1.3 already proved.

**Covers:** FR-10.

### Story 1.5: Save a batch of item changes

As a caller changing several items at once,
I want those writes to commit together,
So that a reorder or a multi-move is never half saved.

**Acceptance Criteria:**

**Given** a body `{ "items": [ ... ] }` and a current `etag` on every object
**When** the caller sends `POST /items/changes`
**Then** the API writes those items in one Cosmos transactional batch in the caller's partition, or writes none (AD-2)
**And** each item still passes the field contract, and the same id, user, and schema rules as Story 1.3 apply

**Given** any missing or mismatched `etag`
**When** the batch is sent
**Then** the API writes nothing. A mismatch returns 409 (AD-6)

**And** the API does not decide membership or order. It stores the `dueDate` and `position` values the caller sent (AD-1, AD-10).

**Notes:**

Where: `POST /items/changes` lands in `ItemsEndpoints`. The write is one Cosmos transactional batch inside `CosmosDocuments`, in the caller's partition. The API still does not decide membership or order. No Scheduling reference, no repository interface, no second data class.

Tools: `Microsoft.Azure.Cosmos` transactional batch on the emulator. The same HTTP client. No new package.

Size: medium to large. The project-structure draft pairs this batch with the etag path as the API work that takes time. It stays inside the types Story 1.1 started.

**Covers:** the write half of FR-12 and FR-15.

### Story 1.6: Create the production stack

As the owner,
I want one production database and API host from the Terraform stack,
So that callers have a single Azure place to use.

**Acceptance Criteria:**

**Given** `bootstrap/` has been applied
**When** the root stack is applied with `prod.tfvars`
**Then** the resources, names, identity, roles, app settings, CORS origins, and outputs are those in `terraform.md` and AD-14 and AD-15
**And** Cosmos is the free-tier account, keys disabled, database `todolist`, container `documents` at 1000 RU/s, partition `/userId`
**And** the Function App has no Cosmos key, no storage connection string, and no Application Insights settings
**And** Terraform outputs the Function App host and the Static Web App host and does not publish the zip or the web files
**And** this story is the Terraform repository. It does not add a project to the application solution (AD-11)

**Notes:**

Where: a separate Terraform repository, not `ToDoList.sln`. `bootstrap/` owns the state account. The root owns the production resource group, Cosmos, and the Static Web App. `modules/function_host` is the only module and owns the plan, the Function App, the user-assigned identity, the function storage account, and the role assignments. Names, settings, and outputs are `terraform.md` and AD-14 and AD-15. This story does not publish the Function App zip or the web files.

Tools: current stable Terraform and the current `hashicorp/azurerm` provider, pinned in `versions.tf` to the versions applied. The account that applies needs permission to create resource groups and role assignments. Cosmos free tier allows one account per subscription.

Size: medium to large the first time this stack is applied. The layout is already specified. The apply order is bootstrap, then the root. It is infrastructure, not application code.

**Covers:** AD-13, AD-14, AD-15.

**Open for the developer:** the globally unique suffix when a storage or Cosmos name is taken. `terraform.md` already says to change bootstrap and `backend.tf` together.

### Story 1.7: Keep local calls off production

As a caller on a developer machine,
I want those calls to use the emulator,
So that local work never writes production items.

**Acceptance Criteria:**

**Given** the API is running on a developer machine
**When** it reads or writes documents
**Then** it uses the emulator connection string from local settings, and those settings are not in Terraform (AD-13)
**And** it does not use the production identity and does not point at the production Cosmos account

**Given** the production Function App from Story 1.6
**When** it reads or writes documents
**Then** it uses the managed identity and `COSMOS_ACCOUNT_ENDPOINT`, through the same concrete Cosmos class (AD-14, AD-16)

**Notes:**

Where: the choice of credential lives in `Program.cs` where `CosmosDocuments` is constructed. Local settings hold the emulator connection string and are gitignored. Production uses `DefaultAzureCredential` with `AZURE_CLIENT_ID` from Story 1.6. Same class either way. No storage interface.

Tools: the emulator container for local, and the Terraform outputs for the production endpoint. `local.settings.json.example` lists setting names and no production values.

Size: small once Stories 1.1 and 1.6 exist. It is a second configuration of one class, not a second data-access stack.

**Covers:** AD-13, AD-14.

## Epic 2: Web

The browser calls the Epic 1 routes. View membership and order are computed in the web scheduling port. This epic does not require Epic 3.

### Story 2.1: Sign in on the web

As a person opening the web app,
I want to sign in with Google before any items are shown,
So that the list I then see is mine.

**Acceptance Criteria:**

**Given** the web app has no Google ID token
**When** a person opens it
**Then** the only surface is Sign In, using the words in `EXPERIENCE.md` (UX-1)
**And** the app does not call `GET /items`

**Given** a token whose audience is `GOOGLE_WEB_CLIENT_ID`
**When** the person is signed in
**Then** the app calls `GET /items` as in Story 1.1 and keeps the items in `itemSession` (AD-3, AD-19)
**And** signing out discards this browser token only and shows no items (FR-2)
**And** colors come from the dark sheet in `tokens.css`, and component classes do not contain hex values (AD-12, UX-10)
**And** `apiBaseUrl` is the Function App host for this build, never a Cosmos connection string (AD-2)

**And** sign-in uses the Google Identity Services script. There is no React Router and no second store (AD-19, AD-20).

**Notes:**

Where: this story creates the web app. Vite project under `web/`, with `src/main.tsx`, `tokens.css`, `app.css`, `auth/googleSignIn.ts`, `api/itemsClient.ts`, `api/problemDetails.ts`, `session/itemSession.ts`, and `presentation/App.tsx` for Sign In versus the signed-in shell. No reference to the C# projects. `googleSignIn.ts` loads the Google Identity Services script and refreshes the token. No React wrapper around that script. No React Router and no Redux.

Tools: Node.js 24, Vite 8.3.5, React 19.3.1, TypeScript 7.0.2. `apiBaseUrl` points at the local Function App. The dark sheet is `tokens.css`. `app.css` uses those variables.

Size: large for the web epic. The project-structure draft is about 22 authored web files. This story is the shell. Board, the form, and drag come later. Google token refresh is part of the setup, not a later story.

**Covers:** FR-1, FR-2, UX-1, UX-10.

### Story 2.2: Add an item on the web

As a person on the web,
I want to type a description and add an item,
So that it is saved and the box is ready for another.

**Acceptance Criteria:**

**Given** a signed-in session and a description that passes `contracts/item-fields.json`
**When** the person submits
**Then** the web validation port checks the fields, then the app sends `POST /items` as in Story 1.2 (AD-7)
**And** the client sets `position` to the end of that due date before the POST. The due date is the day the add was opened on (AD-10, FR-3)
**And** a successful add shows the 201 body in the session and clears the add control
**And** an empty description is not sent

**Given** a 400, a 5xx, or no response
**When** the add fails
**Then** the typed text stays. A 400 shows the field messages. A 5xx or no response shows `Error reaching server` (AD-8, UX-2, UX-5)

**And** the add control and the date field follow `EXPERIENCE.md` Component Patterns. There is no date-picker library (AD-20, UX-3).

**Notes:**

Where: `validation/itemFields.ts` reads `contracts/item-fields.json`. `scheduling/placement.ts` starts here with end-of-day position. `api/itemsClient.ts` gains POST. `presentation/ItemForm.tsx` is the add popup, including the in-app date field: a month grid that starts Monday, typed digits, and an echo of the understood date. No date-picker library. The add control can sit on the signed-in shell until `DayBox` exists in the board stories.

Tools: Vitest 5 for `itemFields.test.ts`. The same Vite app. The field check is TypeScript reading the JSON. It does not call `ToDoList.Core`.

Size: medium to large. The date field is one of the pieces the project-structure draft says takes time, and `ItemForm.tsx` is one of the two large web files. Building the field here means edit reuses it.

**Covers:** FR-3, UX-2, UX-3, UX-5.

### Story 2.3: Change an item's text and date on the web

As a person editing an item in the browser,
I want to change its description, detail, and due date,
So that the saved item replaces the one I had.

**Acceptance Criteria:**

**Given** an item in the session
**When** the person saves a valid edit
**Then** the app sends `PUT /items/` plus the id, as in Story 1.3, with `If-Match` set to the item's `etag`
**And** the date control accepts a calendar month or typed digits, as in `EXPERIENCE.md` (UX-3)

**Given** a 409
**When** the PUT returns
**Then** the session snaps the item back and reloads with `GET /items` (AD-6, UX-5)

**Given** a 400, a 5xx, or no response
**When** the edit fails
**Then** typed text stays, and the error follows Story 2.2

**Notes:**

Where: the edit half of `ItemForm.tsx`, PUT on `itemsClient.ts`, and snapback plus reload inside `itemSession.ts`. The date field from Story 2.2 is the same control. No new library and no new store.

Tools: the Vite app against the local Function App. A 409 is produced by saving the same item from a second caller, then saving with the stale `etag`.

Size: medium if the date field already exists. It fills out the form. It is not a second form.

**Covers:** FR-8, UX-3, UX-5.

### Story 2.4: Change an item's state on the web

As a person working an item on the web,
I want to mark it not started, in progress, or done,
So that the completed date is the date on this browser.

**Acceptance Criteria:**

**Given** the person sets an item to done
**When** the session saves
**Then** it sends `completedDate` as the browser's local calendar date and sends `dueDate` unchanged, through the PUT in Story 2.3 (AD-10, FR-9)

**Given** the person moves the item out of done
**When** the session saves
**Then** `completedDate` is null

**And** the state control follows `EXPERIENCE.md` and calls `itemSession` (UX-3, AD-19).

**Notes:**

Where: the state control on `ItemRow` or `ItemForm`, calling `itemSession`, which sends the existing PUT. Wire values stay `not-started`, `in-progress`, and `done`. No new route.

Tools: the Vite app. No new package.

Size: small. One control and one field on a save that already exists.

**Covers:** FR-9, UX-3.

### Story 2.5: Delete an item on the web

As a person on the web who does not want an item,
I want to confirm before it is deleted,
So that cancel leaves it in place.

**Acceptance Criteria:**

**Given** the person chooses delete
**When** the confirmation is shown
**Then** the words are `Delete this item?` with `Delete` and `Cancel` (UX-2, FR-10)

**Given** the person cancels
**When** the confirmation closes
**Then** no DELETE is sent

**Given** the person confirms
**When** the app sends `DELETE /items/` plus the id
**Then** it follows Story 1.4, including the 409 snapback from Story 2.3

**Notes:**

Where: the Delete link on `ItemForm.tsx`, the confirm words, and DELETE on `itemsClient.ts`. Snapback reuses the session path from edit.

Tools: the Vite app against the local Function App.

Size: small. One action on the form and one method on the HTTP client.

**Covers:** FR-10, UX-2.

### Story 2.6: See Today on the web

As a person on the web,
I want Today to show the open items due today or earlier, and the done items due today,
So that the day matches this browser's date.

**Acceptance Criteria:**

**Given** the items from `GET /items` and the browser's local calendar date
**When** the person opens Today
**Then** the web scheduling port applies the Today membership and sort in AD-10 (FR-4)
**And** the server response is still unfiltered
**And** Today is recomputed when the app returns to the foreground, and an open session does not run a clock (AD-9)
**And** the Today cases live in `contracts/view-examples.json`, and Vitest runs them (AD-18, NFR-3)
**And** empty and loading presentation follow `EXPERIENCE.md` State Patterns (UX-5, UX-11)

**Notes:**

Where: `scheduling/views.ts` and `scheduling/calendar.ts` are the port of the Today rule. `scheduling/views.test.ts` runs the Today cases in `contracts/view-examples.json`. Presentation starts `Board.tsx`, `DayBox.tsx`, and `ItemRow.tsx` in Today mode, including the loading and empty states. The Function App still returns every item. If the example file does not exist yet, this story adds the Today cases. It does not create a second fixture.

Tools: Vitest 5. Vite alias or copy so the test reads `contracts/view-examples.json`. No C# reference.

Size: medium to large. This is where `Board.tsx` starts. The project-structure draft says the view port is small and dense, and that Board grows into the heavy file. Drag and keyboard are later stories on this same file.

**Covers:** FR-4, NFR-3, UX-5, UX-11.

### Story 2.7: Hide completed items on the web

As a person looking at a web view,
I want to hide done items and show them again,
So that the choice sticks in this browser only.

**Acceptance Criteria:**

**Given** a view that contains done items
**When** the person toggles hide completed
**Then** done items leave or return on screen, and no `position` or `dueDate` is sent (AD-9, AD-10, FR-7)
**And** the choice is in localStorage
**And** Today honors it, and Stories 2.8 and 2.9 honor the same flag (UX-4)

**Notes:**

Where: the flag lives in `itemSession.ts` and `localStorage`. `TopBar.tsx` gets the Hide completed link. Views read the flag. No request is sent.

Tools: the Vite app. Browser local storage. No API change.

Size: small. A flag on the session and a word link on the top bar.

**Covers:** FR-7, UX-4.

### Story 2.8: See one date on the web

As a person on the web who picks a date,
I want the items due that day, including done items,
So that I can review it.

**Acceptance Criteria:**

**Given** a chosen calendar date
**When** the person opens that day
**Then** the web scheduling port shows items with that `dueDate`, including done unless Story 2.7 is hiding them (AD-10, FR-5)
**And** the case is in `contracts/view-examples.json` and Vitest runs it
**And** an empty day uses the empty-state words in `EXPERIENCE.md` (UX-2)

**Notes:**

Where: day membership is another function in `scheduling/views.ts`, with a Vitest case in the same example file. `Board.tsx` gains a day mode. `TopBar.tsx` gains the way to pick a date. Same day box as Today.

Tools: Vitest. The Vite app.

Size: small to medium on top of Today. It is another mode of the one board, not a new page.

**Covers:** FR-5, NFR-3.

### Story 2.9: See a week on the web

As a person planning on the web,
I want Monday through Sunday,
So that opening the app lands on the week that contains today.

**Acceptance Criteria:**

**Given** the browser's date and a week start of Monday from one config value
**When** the person opens the app signed in, or switches to the week
**Then** the signed-in opening mode is the week that contains today (FR-14, UX-1)
**And** the web scheduling port shows that Monday through Sunday, earliest day first (AD-9, AD-10, FR-6)
**And** each day is one day box with its own add control, using Story 2.2 (UX-3)
**And** hide completed from Story 2.7 still applies
**And** column counts follow `EXPERIENCE.md` Responsive & Platform (UX-9)
**And** the week cases are in `contracts/view-examples.json` and Vitest runs them

**Notes:**

Where: week membership and the Monday value live in `scheduling/calendar.ts` and `views.ts`. `Board.tsx` renders seven `DayBox` columns and becomes the signed-in opening mode. Column counts are `app.css`, following the experience contract. Each day box reuses the add control from Story 2.2.

Tools: Vitest for the week cases. The Vite app at a wide window and a narrow one.

Size: medium. This finishes the board's layout. Drag is the next large piece, not this one.

**Covers:** FR-6, FR-14, NFR-3, UX-3, UX-9.

### Story 2.10: Reschedule one item on the web

As a person on the web,
I want tomorrow, next week, next month, or a picked date to move one item,
So that it lands at the end of that day.

**Acceptance Criteria:**

**Given** one item and one of those four actions
**When** the person takes it
**Then** the web scheduling port sets `dueDate` and sets `position` to the end of the target date, then saves with the PUT from Story 2.3 (AD-10, FR-11)
**And** next week is the following Monday, and next month is the first of the next month (AD-9)
**And** no other item's `dueDate` changes, and this action does not renumber the day
**And** the placement case is in `contracts/view-examples.json`

**Notes:**

Where: `scheduling/placement.ts` gains append-to-the-end. `scheduling/calendar.ts` answers next week and next month. The buttons are on `ItemForm.tsx`. The save is the existing PUT of one item. A Vitest case covers placement. This action does not call the batch route.

Tools: Vitest. The Vite app.

Size: small to medium. A few functions and the buttons on the form that already exists.

**Covers:** FR-11, NFR-3.

### Story 2.11: Reorder by dragging on the web

As a person arranging a day on the web,
I want to drag an item to a new place, including onto another day,
So that the dropped order is what gets saved.

**Acceptance Criteria:**

**Given** a drag that stays on the same due date, or a drop onto another day
**When** the person drops it
**Then** the web scheduling port sets `position` from the drop index and renumbers that date. A drop onto another day also sets `dueDate` (AD-10, FR-12)
**And** in Today, dropping an overdue item onto today's items sets `dueDate` to the browser's today
**And** the app saves with `POST /items/changes` as in Story 1.5
**And** a 409 snaps back and reloads with `GET /items`
**And** the drag uses pointer events and no drag library (AD-20, UX-6)

**Notes:**

Where: pointer-event drag lives in `Board.tsx`. `scheduling/placement.ts` renumbers the affected day. `itemsClient.ts` gains `POST /items/changes`. A 409 uses the session snapback. No drag library.

Tools: the Vite app and the local Function App. Vitest for the renumber case. Browser pointer events.

Size: large. The project-structure draft says web drag is where the web time sits, together with the keyboard story. This story is the drag half of `Board.tsx`.

**Covers:** FR-12, UX-6.

**Open for the developer:** which sibling items are included in the batch, as long as the batch is all-or-nothing and the saved order matches the drop.

### Story 2.12: Move several items on the web

As a person on the web,
I want to select several items and move them to another date together,
So that one save moves all of them or none.

**Acceptance Criteria:**

**Given** more than one selected item and a target date
**When** the person moves them
**Then** each selected item gets the end-of-day placement from Story 2.10, and the app sends one `POST /items/changes` as in Story 1.5 (FR-15)
**And** no other bulk change exists. State, text, and delete stay one item at a time

**Notes:**

Where: multi-select and the move action live in `Board.tsx`. Placement reuses the end-of-day append. The save reuses the batch call from the drag story. The phone has no equivalent. No other bulk edit.

Tools: the Vite app. Ctrl and click, then one batch request.

Size: medium. Selection plus one save, on top of the drag client. Smaller than building drag.

**Covers:** FR-15.

### Story 2.13: Use the web board from the keyboard

As a person at the web keyboard,
I want the shortcuts in the experience contract,
So that I can add, move, and change items without a pointer.

**Acceptance Criteria:**

**Given** focus is not in a text field
**When** the person uses a shortcut listed in `EXPERIENCE.md` Interaction Primitives
**Then** `itemSession` performs that action through the earlier stories in this epic (UX-7)
**And** the shortcut help surface lists those shortcuts
**And** presentation keeps the accessibility floor in `EXPERIENCE.md` (UX-8)
**And** no shortcut adds a route, a second store, or a component kit (AD-19, AD-20)

**Notes:**

Where: the key handlers live in `Board.tsx` and call `itemSession`. They do not add a route or a second store. `ShortcutHelp.tsx` is the `?` list. `TopBar.tsx` gains the Shortcuts link. The accessibility floor, focus order, names, contrast on the dark tokens, and reduced motion, applies to the controls this epic already has. No component kit.

Tools: the Vite app. Keyboard only, with focus outside a text field. Vitest is optional here. The behavior is the session methods the earlier stories already have.

Size: large beside the other web stories. The project-structure draft pairs the keyboard work with drag as the heavy part of the 22-file web app. Many keys, one board, plus the help surface.

**Covers:** UX-7, UX-8.

## Epic 3: Android

The phone calls the Epic 1 routes. View membership and order are computed in `ToDoList.Scheduling`. This epic does not require Epic 2. xUnit runs `contracts/view-examples.json`. If a case is already there, this epic uses it. If this epic is built first, it adds the case. It does not add a second example file.

### Story 3.1: Sign in on the phone

As a person opening the Android app,
I want to sign in and sign out on this phone,
So that this device keeps its own Google token.

**Acceptance Criteria:**

**Given** the Android app
**When** a person opens it
**Then** it opens on Add new, and Sign In is a text link on that screen (FR-14, UX-1)
**And** it does not call `GET /items` until this client has a token

**Given** a token whose audience is `GOOGLE_ANDROID_CLIENT_ID` for the Play Store package
**When** the person is signed in
**Then** the app calls `GET /items` as in Story 1.1 and keeps the items in `ItemSession` (AD-3, AD-19)
**And** signing out discards this phone's token only and shows no items (FR-2)
**And** colors use the same token names as the web dark sheet, in a resource dictionary, with no theme picker (AD-12, UX-10)
**And** sign-in uses Credential Manager (AD-20)
**And** `apiBaseUrl` is the Function App host for this build, never a Cosmos connection string (AD-2)

**Notes:**

Where: this story creates `ToDoList.App` from `dotnet new maui` and publishes Android only. Leave the iOS, Mac Catalyst, and Windows folders the template adds. Unimplemented. Auth is `Api/GoogleSignIn.cs` through Credential Manager. The project-structure draft assumes the server client id passed to Credential Manager is the web client id. The Android OAuth client still matches the Play Store package. `Api/ItemsClient.cs` does the GET. `Session/ItemSession.cs` holds the items. `Presentation/AddNewPage` is the opening screen and the Sign In / Sign Out links. `Resources/Styles/Colors.xaml` is the dark token set. `MauiProgram.cs` wires those types. The app references Core when later stories need the field check. This story does not need Scheduling yet.

Tools: .NET MAUI 10.0.120, `Xamarin.Google.Android.Libraries.Identity.GoogleId` 1.1.0.16, and an Android emulator or device. `apiBaseUrl` is the local Function App. No MVVM toolkit.

Size: large. The project-structure draft puts the MAUI app at about 16 authored files and names Google sign-in as one of the two heavy parts. This story is the app skeleton plus that sign-in.

**Covers:** FR-1, FR-2, FR-14, UX-1, UX-10.

**Open for the developer:** the Play Store package name. Architecture fixes the match to `GOOGLE_ANDROID_CLIENT_ID`, not the package string.

### Story 3.2: Add an item on the phone

As a person on the phone,
I want to type a description and add it, including when I was signed out,
So that capture stays on one screen.

**Acceptance Criteria:**

**Given** the phone is signed out and Add new is showing
**When** the person enters a description and presses Add
**Then** the typed text stays and the app asks them to sign in, with the words `Sign In to save this item.` (UX-1, UX-2)
**And** after this client has a token, the app sends `POST /items` as in Story 1.2

**Given** the phone is already signed in
**When** the person adds a valid description
**Then** `ToDoList.Core` checks the fields, `ToDoList.Scheduling` sets `position` to the end of that due date, and the POST stores the item (AD-7, AD-10, AD-17)
**And** the box clears and the session shows the 201 body
**And** a failed add keeps the text and uses the same error words as the web (AD-8, UX-5)

**And** the app still opens on Add new. There is no local database (FR-14).

**Notes:**

Where: `AddNewPage` is the description and Add. `ItemSession` calls `ItemRules` in Core before send, and `Placement` in `ToDoList.Scheduling` for the end position. This story creates the Scheduling project with `Placement.cs` only. Today and week come later. `ItemsClient` gains POST. Signed out, the page keeps the text and asks for sign-in. There is no local database.

Tools: the MAUI Android app against the local Function App. `ItemRules` reads `contracts/item-fields.json`. xUnit can cover Placement's append when the test project starts. The full date field is the edit page, not this capture screen.

Size: medium. Add new is a form. Scheduling at this point is one small type, not the view engine.

**Covers:** FR-3, UX-1, UX-2, UX-5.

### Story 3.3: Change an item's text and date on the phone

As a person editing an item on the phone,
I want to change its description, detail, and due date,
So that the saved item replaces the one I had.

**Acceptance Criteria:**

**Given** an item in `ItemSession`
**When** the person saves a valid edit from the full-screen edit surface
**Then** the app sends the PUT from Story 1.3 with `If-Match`
**And** the date field is written in the app, with no date-picker library (AD-20, UX-3)

**Given** a 409
**When** the PUT returns
**Then** the session snaps back and reloads with `GET /items` (AD-6, UX-5)

**Notes:**

Where: `Presentation/EditItemPage` is the full-screen form. The date field is written in the app: a month grid that starts Monday, typed digits, and an echo of the understood date. No date-picker library. `ItemsClient` gains PUT. `ItemSession` snaps back and reloads on 409.

Tools: the MAUI Android app. The same field behavior as the web date field, implemented here in XAML and C#.

Size: large for the phone. The project-structure draft names the in-app date field as one of the places time concentrates. This story is that field plus the edit page. Later phone stories add actions to this page rather than a second editor.

**Covers:** FR-8, UX-3, UX-5.

### Story 3.4: Change an item's state on the phone

As a person working an item on the phone,
I want to mark it not started, in progress, or done,
So that the completed date is the date on this phone.

**Acceptance Criteria:**

**Given** the person sets an item to done, including by the state box or a double-tap
**When** the session saves
**Then** it sends `completedDate` as the phone's local calendar date and sends `dueDate` unchanged, through the PUT in Story 3.3 (AD-10, FR-9, UX-3)

**Given** the person moves the item out of done
**When** the session saves
**Then** `completedDate` is null

**Notes:**

Where: the state box and the double-tap live on the item row, calling `ItemSession`, which sends the PUT from edit. The three states are the wire values. No new route and no new library.

Tools: the MAUI Android app.

Size: small to medium. The interaction is specified. The work is the control and the gesture, on the session that already saves.

**Covers:** FR-9, UX-3.

### Story 3.5: Delete an item on the phone

As a person on the phone who does not want an item,
I want to confirm before it is deleted,
So that cancel leaves it in place.

**Acceptance Criteria:**

**Given** the person chooses delete
**When** the confirmation is shown
**Then** the words are `Delete this item?` with `Delete` and `Cancel` (UX-2, FR-10)

**Given** the person confirms
**When** the app sends DELETE
**Then** it follows Story 1.4, including the 409 snapback from Story 3.3

**Given** the person cancels
**When** the confirmation closes
**Then** no DELETE is sent

**Notes:**

Where: the Delete link on `EditItemPage`, the confirm words, and DELETE on `ItemsClient`. Cancel sends nothing. Snapback reuses the session.

Tools: the MAUI Android app.

Size: small. One link on the edit page and one method on the HTTP client.

**Covers:** FR-10, UX-2.

### Story 3.6: See Today on the phone

As a person on the phone,
I want Today to show the open items due today or earlier, and the done items due today,
So that the day matches this phone's date.

**Acceptance Criteria:**

**Given** the items from `GET /items` and the phone's local calendar date
**When** the person opens Today
**Then** `ToDoList.Scheduling` applies the Today membership and sort in AD-10 (FR-4)
**And** the server response is still unfiltered
**And** Today is recomputed when the app returns to the foreground (AD-9)
**And** xUnit runs the Today cases in `contracts/view-examples.json` (AD-18, AD-21, NFR-3)
**And** empty and loading presentation follow `EXPERIENCE.md` State Patterns (UX-5)

**Notes:**

Where: `ToDoList.Scheduling` gains `Views.cs` for Today. `ToDoList.Tests` gains `ViewExampleTests`, which loads `contracts/view-examples.json` and does not reference the API. `Presentation/BoardPage` starts here as the Today mode, including loading and empty. The app still opens on Add new. If the example file has no Today cases yet, this story adds them. It does not create a second fixture.

Tools: xUnit 3. The MAUI Android app for the page. The emulator is not a test fixture.

Size: medium. This introduces `BoardPage` and the view rule. The project-structure draft says the heavy part of that page is the gestures, which are the later drag story. Today is membership, sort, and the first mode.

**Covers:** FR-4, NFR-3, UX-5.

### Story 3.7: Hide completed items on the phone

As a person looking at a phone view,
I want to hide done items and show them again,
So that the choice sticks on this phone only.

**Acceptance Criteria:**

**Given** a view that contains done items
**When** the person toggles hide completed
**Then** done items leave or return on screen, and no `position` or `dueDate` is sent (AD-9, AD-10, FR-7)
**And** the choice is in Android Preferences
**And** Today honors it, and Stories 3.8 and 3.9 honor the same flag (UX-4)

**Notes:**

Where: the flag is in `ItemSession` and Android Preferences. The link is on the board top bar. Views honor it. No API call.

Tools: the MAUI Android app. Android Preferences. No new package.

Size: small. A device flag and a word link.

**Covers:** FR-7, UX-4.

### Story 3.8: See one date on the phone

As a person on the phone who picks a date,
I want the items due that day, including done items,
So that I can review it.

**Acceptance Criteria:**

**Given** a chosen calendar date
**When** the person opens that day
**Then** `ToDoList.Scheduling` shows items with that `dueDate`, including done unless Story 3.7 is hiding them (AD-10, FR-5)
**And** xUnit runs that case from `contracts/view-examples.json`
**And** an empty day follows the phone empty state in `EXPERIENCE.md`

**Notes:**

Where: day membership is `Views.cs`. `BoardPage` gains the day mode. Portrait shows Add new and one day, one visible at a time, with a horizontal scroll between them. xUnit runs the day case from the shared example file.

Tools: xUnit. The MAUI Android app in portrait and landscape.

Size: medium. The day mode is still `BoardPage`, and the column scroll starts here. Pick-up and reorder are the later gesture story.

**Covers:** FR-5, NFR-3.

### Story 3.9: See a week on the phone

As a person planning on the phone,
I want Monday through Sunday,
So that I can switch to the week without losing Add new as the opening screen.

**Acceptance Criteria:**

**Given** the phone's date and the same Monday config value
**When** the person switches to the week
**Then** `ToDoList.Scheduling` shows that Monday through Sunday, earliest day first (AD-9, AD-10, FR-6)
**And** the app still opens on Add new, as Story 3.2 left it (FR-14)
**And** each day is one day box, and the column counts follow `EXPERIENCE.md` Responsive & Platform (UX-3, UX-9)
**And** hide completed from Story 3.7 still applies
**And** xUnit runs the week cases in `contracts/view-examples.json`
**And** there is one session and no second store (AD-19)

**Notes:**

Where: `WeekStart.cs` holds Monday behind one property. `Views.cs` gains the week. `BoardPage` shows Add new plus seven day columns and the wider layouts from the experience contract. Opening the app stays on Add new. xUnit runs the week cases.

Tools: xUnit. The MAUI Android app at phone and small-tablet widths.

Size: medium. This is the rest of `BoardPage`'s layout. The pick-up drag is still the large gesture story.

**Covers:** FR-6, FR-14, NFR-3, UX-3, UX-9.

### Story 3.10: Reschedule one item on the phone

As a person on the phone,
I want tomorrow, next week, next month, a picked date, or a swipe to the next or previous day,
So that one item lands on that day.

**Acceptance Criteria:**

**Given** one item and one of those actions
**When** the person takes it
**Then** `ToDoList.Scheduling` sets `dueDate` and sets `position` to the end of the target date, then the app saves with the PUT from Story 3.3 (AD-10, FR-11)
**And** a horizontal swipe while the item is picked up moves it one day, as in `EXPERIENCE.md` Interaction Primitives
**And** next week is the following Monday, and next month is the first (AD-9)
**And** no other item's `dueDate` changes
**And** xUnit runs the placement case in `contracts/view-examples.json`
**And** several items cannot be selected (FR-13)

**Notes:**

Where: `Calendar.cs` answers next week and next month. `Placement` appends the item. The buttons are on `EditItemPage`. A picked-up swipe of one day is the gesture in the experience contract, so this story starts that pick-up on `BoardPage` and saves with the existing PUT. `ToDoList.Tests` can add `PlacementTests`. There is no multi-select.

Tools: xUnit for placement. The MAUI Android app.

Size: medium. The date actions are small. The one-day swipe is the start of the gesture code that reorder extends. It is not the full drag.

**Covers:** FR-11, NFR-3.

### Story 3.11: Reorder by dragging on the phone

As a person arranging a day on the phone,
I want to drag a picked-up item within the day or onto another day,
So that the dropped order is what gets saved.

**Acceptance Criteria:**

**Given** a picked-up item dragged within the day, or onto another day
**When** the person drops it
**Then** `ToDoList.Scheduling` sets `position` from the drop index and renumbers that date. A drop onto another day also sets `dueDate` (AD-10, FR-12)
**And** in Today, dropping an overdue item onto today's items sets `dueDate` to the phone's today
**And** the app saves with `POST /items/changes` as in Story 1.5
**And** a 409 snaps back and reloads with `GET /items`
**And** the gesture is the phone pick-up and drag in `EXPERIENCE.md`. There is no drag library (AD-20, UX-6)

**Notes:**

Where: the rest of the pick-up lives on `BoardPage`: drag up or down within the day, and across a day. `Placement` renumbers. `ItemsClient` gains `POST /items/changes`. Snapback reuses the session. No drag library.

Tools: the MAUI Android app against the local Function App. xUnit for the renumber. Touch gestures, not a package.

Size: large. The project-structure draft says phone gestures are where the MAUI time sits. This story is that work, on the `BoardPage` the view stories started.

**Covers:** FR-12, UX-6.
