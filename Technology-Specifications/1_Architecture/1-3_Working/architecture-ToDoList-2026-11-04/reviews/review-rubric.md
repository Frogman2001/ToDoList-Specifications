# Rubric review — Architecture Spine (ToDoList)

Reviewer: rubric walker (good-spine checklist).
Content: `ARCHITECTURE-SPINE.md` (feature altitude, client-computed views).
Checked: 2026-11-04. Spine not modified.

**Verdict: Revise before handoff.** The spine fixes the real paradigm, write boundary, document shape, schema, and Azure envelope, and it does not contradict the brief, PRD, or UX. It is not yet a contract two epics can build against: the shared HTTP contract is missing, two rules do not prevent the divergence they name, one Deferred item still lets the clients diverge, and Android release — already adopted — is silent.

## Checklist

| Check | Result |
| --- | --- |
| Fixes the real divergence points for the level below, and misses none | **Fail.** Write path, document shape, schema, etag-on-mismatch, field lengths, week bounds, solution shape, tokens, and the two-environment Azure target are fixed. The shared request contract, view membership and sort, position uniqueness, and Android distribution are not. |
| Every AD Rule is enforceable and actually prevents its stated divergence | **Fail.** AD-6 and AD-10 do not. AD-2's single transactional batch cannot be carried out past Cosmos's batch ceiling. The other rules match their Prevents clauses. |
| Nothing under Deferred could let two units diverge | **Fail.** One item can. The rest are real non-goals or are already bound by an AD. |
| Named tech is verified-current | **Pass**, with one low precision note (Node.js). Versions were checked against current vendor docs on 2026-11-04; see Versions. |
| Ratifies rather than contradicts inputs | **Pass** on the six source documents: open technical questions in those docs are settled, not reversed. **Fail** on one adopted run decision: Play Store publication is not in the spine. |
| Every dimension this altitude owns is decided, deferred, or an open question | **Fail** for one slice of deployment. Environments, the Azure product choices, and operations are decided or explicitly deferred. They are not silent. |

## Versions

Checked 2026-11-04. These named rows are real and still fit.

| Named | What is current | Spine |
| --- | --- | --- |
| .NET 10 | LTS, Functions isolated support through 2028-11-14. In-process support ends 2026-11-10. .NET 10 is not on Linux Consumption. | Matches AD-13 and the stack row. |
| .NET MAUI 10.0.120 | Latest patch, released 2026-10-21. Support policy (updated 2026-10-22): MAUI 10 supported through 2027-05-11. MAUI 9 is out of support. | Exact pin matches. |
| Azure Functions, Flex Consumption, isolated worker, .NET 10 | Microsoft Learn lists C# isolated .NET 8, 9, and 10 on Flex Consumption. In-process is not supported there. | Matches. |
| React 19 | Current major. React 19.3 shipped 2026-09-09. | Major pin is current. |
| Vite 8 | Vite 8.0 shipped 2026-03-12 (Rolldown). 8.3.x is current as of 2026-10-21. Requires Node.js 20.19+ or 22.12+. | Major pin is current. |
| Node.js 22 | Maintenance LTS through 2027-04-30. Still supported. Active LTS is Node.js 26. | Supported, not the newest LTS. Vite 8 rejects Node 22 below 22.12. Low note R-L1. |
| Cosmos DB, API for NoSQL, free tier | Learn: first 1000 RU/s and 25 GB free for the life of the account, one free-tier account per subscription, chosen at creation. Not available on serverless. | Matches the stack row and AD-13. |
| Cosmos emulator `vnext-latest` | GA image documented 2026-06-02 (`mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest`). A pre-GA etag/batch bug was fixed in `vnext-EN20260430` (2026-05-18), before GA. | Tag is the documented current image. |
| Azure Static Web Apps | Still the documented host for a Vite/React static build. "current React host" is not a release pin. The product has no library version to pin. | Accepted. Not a finding. |

AD-2's choice (browser calls the Function App URL; the Static Web App serves files only) fits a separate Flex Consumption app. Static Web Apps' managed Functions host is a different, narrower runtime and is correctly not used.

## What holds

- AD-1, AD-4, and AD-5 stop a stored Today list, a single items blob, and an all-optional document class.
- AD-3 stops a shared session, a second identity provider, and email-as-id. Logout is per client. The user id is the Google `sub`.
- AD-7, AD-8, AD-9, AD-11, AD-12, and AD-13 are enforceable and match the decisions adopted in the run: field lengths, problem-details copy, Monday week, device-only hide-completed, foreground recompute, project list, dark tokens, free-tier production account, emulator for local.
- The capability map covers FR-1 through FR-15 and NFR-1 through NFR-3. Coverage is not the same as a rule that actually binds each row (R-L2).
- Deferred items other than idempotency keys are safe. Date-window queries are already forbidden by AD-1. Per-user week start, a phone theme picker, a batch migration, a second Azure environment, always-ready instances, and a custom telemetry product are already bound or are explicit non-goals. The deployment pipeline can wait: AD-13 already fixes the production target.
- Environments (local and production), infra products (Static Web App, Flex Consumption, one free-tier Cosmos account), and operations (platform logs, no custom telemetry product, scale to zero) are decided or deferred. The operational envelope is not a silent dimension.

No source document is contradicted. The frozen PRD addendum leaves item embedding and the stack open; the spine settles them. The frozen experience row leaves hide-completed persistence to the build; AD-9 settles it on the device. Those are decisions the sources asked for.

---

## Findings

### R1 — High — Shared HTTP contract is missing

- **Location:** AD-2, AD-3, AD-6, AD-8; Structural Seed; Consistency Conventions
- **Trigger:** Web, Android, and the API can each obey every AD and still not accept one another's calls.
- **What diverges:** Resource paths and verbs; JSON property names and casing; where the etag travels (header versus body); what create returns (id, etag, schema version); whether the browser calls the Function App cross-origin or a Vite/`/api` proxy; which origin CORS allows. AD-2 requires both clients to call the Function App URL and forbids the phone using the Static Web App for data, so the browser call is cross-origin in production and across ports locally. Nothing states that.
- **Guard:** Add one AD, or one `contracts/` document the AD names, that pins camelCase JSON, `Authorization: Bearer`, etag on `If-Match`, the create response body, CORS for the Static Web App origin and the local Vite origin, and the resource set (list every item in the caller's partition, create, update, delete, and one batch change for reorder and web multi-move). State that every item route requires the bearer token.
- **Consequence:** Independent epics ship two clients that cannot share one API, or one client sneaks data through the Static Web App and violates AD-2.
- **Disposition:** Autofix.

### R2 — High — AD-10 does not lock the view result it claims to lock

- **Location:** AD-1 (line 52), AD-9 (line 101), AD-10 (lines 105–107), conventions row "Data and formats"
- **Trigger:** Both clients compute "from the device date," both run the shared examples, and the views still differ.
- **What diverges:** The rule's Prevents clause is "the web app and the phone ordering or completing an item differently." Completing is specified. Ordering is not. The examples are required to state items, a device date, and "the resulting order." They are not required to state which items are in Today, a day, or a week. The PRD predicates are absent: Today is open items due on or before today plus done items due today; overdue open items precede today, oldest due date first; within a due date, position ascending; a specific date is items due that date, including done; a week is items whose due date falls in that Monday–Sunday. "Device date" is not defined as the device's local calendar date, so one client can use UTC and the other local time and both pass a fixture that is handed an already-chosen date. Positions are "the integers the client submits," with no uniqueness rule per due date, so ties sort differently on each client. A new item "goes to the end" is in the PRD and not in the rule.
- **Guard:** Put those predicates, the sort key, local-calendar-date (not UTC), and unique integer positions per due date into the AD-10 rule. Keep the shared examples as the executable check, and say they cover membership as well as order.
- **Consequence:** NFR-3 fails while every AD is observed. This is the divergence the paradigm exists to prevent.
- **Disposition:** Autofix.

### R3 — High — A missing etag still allows last-write-wins

- **Location:** AD-6 (lines 81–82)
- **Trigger:** An update or delete omits `_etag`.
- **What diverges:** The rule says updates and deletes send the etag and a mismatch returns 409. A missing etag is not a mismatch. An API that then overwrites, and a client that omits the etag, both follow the letter of the rule. That is the silent last-write-wins the Prevents clause names. The same hole is the response half of R1: if create does not return an etag, the next save cannot send one.
- **Guard:** A missing etag on update or delete is rejected (400 or 409) and writes nothing. Only a matching etag writes. Say so in the rule.
- **Consequence:** The phone and the web overwrite each other with no 409 and no snap-back, which is the case AD-6 was adopted to stop.
- **Disposition:** Autofix.

### R4 — High — Android release is a silent deployment decision

- **Location:** AD-13 (lines 122–125); Deferred pipeline bullet (line 186); absent elsewhere
- **Trigger:** The run adopted "the Android app is published to the Play Store." The spine never decides, defers, or questions how the MAUI app is released.
- **What diverges:** Web and API production targets are fixed. The third runtime can be sideloaded, put on an internal track, or shipped to Play production, with different package identity and signing than the Google Android client id in AD-3. Those choices change auth and what "deployed to production" (PRD SM-1) means. This is the deployment slice the checklist treats as a finding when it is neither decided nor deferred.
- **Guard:** Record the adopted target on AD-13 or in the structural seed: project 1 ships the Android app through the Play Store. Leave pipeline mechanics in Deferred. Tie the Android OAuth client to that package (see R-M6).
- **Consequence:** "In production" is true for the site and false or ambiguous for the phone, and the Google audience in AD-3 may not match the binary that ships.
- **Disposition:** Autofix. The decision is already adopted; the spine dropped it.

### R-M1 — Medium — One transactional batch is not always possible

- **Location:** AD-2 (line 58)
- **Trigger:** A reorder or a web multi-move touches more than one Cosmos transactional batch can hold.
- **What diverges:** Learn, current limits: 100 operations per transactional batch and a 2 MB request. The rule says the API writes every changed item in one batch or writes none. It does not say what happens when one batch cannot be formed. One unit can split the write and break the all-or-nothing rule. Another can fail the request. Both can claim the rule.
- **Guard:** If the change does not fit one batch, the API writes none and returns an error. It does not split. Clients do not send a partial reorder.
- **Consequence:** A large reorder or multi-move is atomic on one implementation and partial on another. Unlikely at hobby size; the rule as written still cannot be followed past the cap.
- **Disposition:** Autofix.

### R-M2 — Medium — Deferred idempotency keys let the clients diverge

- **Location:** Deferred (line 183)
- **Trigger:** A retried add (timeout after the server has saved).
- **What diverges:** "Client-generated idempotency keys for retried adds" pushes the mechanism down without a negative rule. One client invents a key. The other retries a bare create and duplicates the item. AD-7 does not say creates are not idempotent.
- **Guard:** State in an AD, and remove or reword the Deferred bullet: project 1 has no idempotency key; a retried create may insert a second item; neither client invents a key.
- **Consequence:** The same failed add yields one item on one client and two on the other.
- **Disposition:** Autofix.

### R-M3 — Medium — "Accepts a bearer token" does not require that the token be verified

- **Location:** AD-3 (line 64); AD-4 partition key (line 71)
- **Trigger:** The API reads the `aud` and `sub` claims without checking the signature, issuer, or expiry, or it trusts a `userId` in the body.
- **What diverges:** The stated Prevents clause (shared session, second provider, email as id) is covered. The rule does not stop a decode-only implementation, which still "accepts a bearer token whose audience is" one of the two client ids. Partition `userId` is "equal to the Google `sub`" but is not required to be taken only from the verified token.
- **Guard:** The API verifies signature, issuer, expiry, and audience (web client id or Android client id). The caller id is that token's `sub`. A body or query value cannot select another partition.
- **Consequence:** NFR-1 fails on a literal reading of AD-3. Another user's items are readable if the API trusts the request.
- **Disposition:** Autofix.

### R-M4 — Medium — Scheduling's dependency on Core is undecided

- **Location:** Dependency diagram (lines 30–44); AD-11 (line 113)
- **Trigger:** Two builders place the item type.
- **What diverges:** The diagram is a rule. It shows API → Core and MAUI → Core and MAUI → Scheduling. It does not show Scheduling → Core, and it does not forbid Core → Scheduling. One epic puts the item type in Core and references it from Scheduling. Another copies the type into Scheduling so the web JSON and the phone library drift in shape even when the view examples match.
- **Guard:** Scheduling may reference Core. Core does not reference Scheduling. Web references neither, and keeps using the shared JSON fixtures (already in AD-11).
- **Consequence:** Two item shapes, or view rules leaking into Core beside the phone library.
- **Disposition:** Autofix.

### R-M5 — Medium — Shared token names have no source

- **Location:** AD-12 (lines 117–118)
- **Trigger:** Web and MAUI each ship a dark theme that uses variables.
- **What diverges:** The rule requires the same names and forbids a hard-coded color and a phone theme picker. It does not say the names come from `DESIGN.md` (`colors.background`, `colors.surface`, `colors.line`, `colors.text`, `colors.text-muted`, `colors.accent`, `colors.surface-picked`, `colors.error`, `rounded.sm`). Each client can invent a coherent dark sheet and satisfy the rule.
- **Guard:** Name `DESIGN.md` as the token source for project 1. Both `tokens.css` and the MAUI resource dictionary use those names. Values stay the dark sheet.
- **Consequence:** The theme swap AD-12 is written to protect has nothing stable to swap. The clients look like two products.
- **Disposition:** Autofix.

### R-M6 — Medium — Create-time infra choices are neither decided nor deferred

- **Location:** AD-13; Stack; Deferred pipeline bullet
- **Trigger:** The first Azure resources and the Google clients are created.
- **What diverges:** Products are chosen. Region is not, and it is not in the pipeline deferral. Cosmos free tier is single-account and the unpaid allowance is per account; a second region is a different bill and a different failure mode. Separately, nothing says there is one Google project with exactly two OAuth client ids (web and Android) matching AD-3's audiences. Two units can create two Google projects, or an Android client whose package does not match the Play binary (R4).
- **Guard:** Defer the region with the pipeline, and bind the constraint now: one Azure region, chosen when the first resources are created; Cosmos stays single-region. Add the Google constraint now: one project, two client ids, audiences identical to AD-3.
- **Consequence:** The free-tier account and the Function App are created in incompatible or costly layouts, or sign-in works on only one client.
- **Disposition:** Defer the region (with the constraint written down). Autofix the Google project rule.

### R-M7 — Medium — Due date is a format, not a required field

- **Location:** AD-7 (line 88)
- **Trigger:** A client omits due date on create.
- **What diverges:** Description is required. Due date is specified only as `YYYY-MM-DD`. The glossary says due date is required and defaults to today, and that it is what drives display. One client defaults it. Another omits it. Core is required to trust the value after the API check, so the API must be the one that rejects an absent or unparseable due date.
- **Guard:** In AD-7, due date is required, `YYYY-MM-DD`, no time zone. Completed date is absent unless state is `done`, and is cleared when state leaves `done` (AD-10 already says the transition; the field rule should match).
- **Consequence:** Items with no due date have no defined Today, week, or order. The view rules in R2 cannot be applied.
- **Disposition:** Autofix.

### R-L1 — Low — Node.js 22 is broader than Vite 8 allows

- **Location:** Stack, Node.js row (line 143)
- **Trigger:** A machine or CI image installs Node 22.0 through 22.11.
- **Guard:** Pin the floor Vite 8 documents: 22.12 or newer (20.19+ is the other supported line; 22.12+ matches this row).
- **Consequence:** The web project fails to start on a Node 22 that the stack row permits.
- **Disposition:** Autofix.

### R-L2 — Low — FR-14 is mapped to rules that do not govern it

- **Location:** Capability map (line 173)
- **Trigger:** A builder treats the Governed-by cell as the full constraint for opening screens and signed-out entry.
- **What diverges:** FR-14 (web opens on the week that contains today; Android opens on Add new) and the experience rules (signed-out web gates; signed-out Android keeps the typed add and then asks to sign in) are not in AD-2 or AD-10. Those behaviors are already specified in the PRD and `EXPERIENCE.md`. They do not need a new architectural trade-off. The map cell overclaims.
- **Guard:** Point that row at FR-14 and the experience signed-out rows, and keep AD-2 only for the multi-move request.
- **Consequence:** An epic that reads only the map invents a second opening screen or a second signed-out path.
- **Disposition:** Autofix the map cell.

## Deferred items that do not fail

These do not let two project-1 units diverge, because an AD already forbids the alternative or the item is a product non-goal:

- iOS, extra identity providers, sharing, offline, admin tools.
- Date-window queries and paging (AD-1 returns the whole partition and does not filter).
- Per-user week start (AD-9 locks Monday).
- Phone theme picker (AD-12 ships dark only).
- Batch migration (AD-5 converts on read).
- A second Azure environment, always-ready instances, and a custom telemetry product (AD-13 plus this bullet).
- The deployment pipeline, given the production target in AD-13. It does not cover Android distribution (R4) or region (R-M6).

## Not a finding

- Restating keyboard shortcuts, delete confirmation, or layout columns. The experience document already binds those, and they are not architectural forks.
- Mermaid in the spine. The adopted D2 diagrams belong to the human review document, not to this substrate.
- Major-only pins for .NET, React, and Vite. The majors were checked and are current. MAUI is pinned to the latest patch.
- Static Web Apps named as "current React host." The service is current and has no package version. AD-2 correctly avoids the managed Functions runtime.
