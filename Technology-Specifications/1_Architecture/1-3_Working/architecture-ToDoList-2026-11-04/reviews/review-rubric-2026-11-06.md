# Rubric review — ToDoList architecture spine

- **Spine:** `ARCHITECTURE-SPINE.md` (updated 2026-11-06)
- **Lens:** good-spine checklist (rubric walker)
- **Date:** 2026-11-06
- **Altitude judged:** feature (web client, Android client, one back end)
- **Verdict:** **Fail**

AD-13, AD-14, and AD-15 do not contradict AD-1 through AD-12. They decide deployment, environments, and the Azure resource shape that a domain-only spine would have left silent. The spine still fails because two host contracts are unnamed, so the web client, the Android client, and the Terraform stack can ship incompatible edges, and because backup is neither a rule nor a Deferred item.

This review read the spine only. It did not re-open the brief, PRD, UX, or companion Terraform docs. This repository has no application code, so the brownfield check is vacuous.

## Checklist

| Criterion | Result |
| --- | --- |
| Fixes the real divergence points one level down, and misses none | Fail. Product rules (facts vs views, single writer, document shape, etag, field contract, view parity, solution shape, tokens) are fixed. Host auth and CORS headers, who writes Function App settings, and the drag write path are not. |
| Every AD Rule is enforceable and prevents its stated divergence | Fail for the new host rules. AD-1–AD-12 and most of AD-13–AD-15 do prevent what they name. AD-13’s CORS sentence and AD-14’s setting list do not pin the facts a second implementer needs. |
| Nothing under Deferred can let two units diverge | Pass with one soft spot. The idempotency bullet does not say “not in project 1.” The other bullets either exclude the work or restate a rule already adopted. |
| Named tech is verified-current | Pass, with patch nits. Checked 2026-11-06. See Versions. |
| Ratifies rather than contradicts a brownfield codebase | Pass. Spec repo; no app code to ratify or contradict. The spine is a greenfield build substrate. |
| Inherited parent spine is not weakened | Not applicable. This is the feature spine. No inherited AD list. |
| Every dimension this altitude owns is decided, deferred, or an open question | Fail. Deployment, environments, and provider strategy are decided. Operations is only half spoken: metrics are decided, backup and alerting are silent. |

## AD-13, AD-14, and AD-15 against AD-1–AD-12

No new rule reverses an earlier one.

- **Stored facts and the only writer (AD-1, AD-2).** Production and local both go through the Function App. The Static Web App serves files only. The emulator is local-only. Cosmos names stay `todolist` / `documents`. Session consistency and a single-region free-tier account do not fight the partition batch or the etag rule.
- **Identity (AD-3).** `GOOGLE_WEB_CLIENT_ID` and `GOOGLE_ANDROID_CLIENT_ID` are the same setting names in AD-14. AD-13 binds the Android client id to the Play Store package. No second identity provider appears.
- **No Cosmos key in Azure (AD-14) vs the local emulator string.** The Prevents line is the Function App in Azure. The Rule then allows the emulator connection string in local settings and says that machine does not use the managed identity. That is a scoped exception, not a second production path.
- **“One Azure resource group” (AD-13) vs the Terraform state account (AD-15).** AD-13 inventories the production app group. AD-15 puts remote state in a storage account and resource group the root stack does not create. That account is bootstrap, not a second app environment. It does not violate the production inventory. It is also not named (see Low).
- **“An infrastructure project” (AD-11 Prevents) vs the Terraform repository (AD-15).** The AD-11 Rule is a .NET shape: Cosmos is a concrete class in the API, and an interface exists only when two implementations exist. The Terraform repo is a different repository, already required by AD-15. The word “infrastructure project” in the Prevents line can be misread as forbidding that repo. The Rule does not forbid it. Wording only.
- **Deferred lines that repeat AD-13 and AD-14.** “A second Azure environment and always-ready Function instances” and “Application Insights, a Log Analytics workspace, Google logging, and usage rows in Cosmos” restate adopted prohibitions. They do not reopen them. The release-pipeline bullet matches AD-15: Terraform creates empty resources and outputs hosts; it does not publish the zip or the web files.
- **Local machine.** AD-13 says Terraform does not describe the local machine. AD-14 says the developer API uses the emulator string. Same split.

The gap is ownership, not opposition. AD-14 lists the production setting names. AD-15’s module list never includes those settings. The deferred pipeline publishes artifacts, not configuration. Nothing in AD-1–AD-12 conflicts with that split; the split itself is unfinished.

## Findings

### High

**1. Nobody owns the Function App setting values.** Disposition: **autofix**.

AD-14’s Rule requires these settings on the Function App: `AzureWebJobsStorage__accountName`, `AzureWebJobsStorage__credential` = `managedidentity`, `AzureWebJobsStorage__clientId`, `AZURE_CLIENT_ID`, `COSMOS_ACCOUNT_ENDPOINT`, `COSMOS_DATABASE_NAME`, `COSMOS_CONTAINER_NAME`, `GOOGLE_WEB_CLIENT_ID`, and `GOOGLE_ANDROID_CLIENT_ID`. AD-3 and AD-4 only work if those values are the managed-identity client id, the Cosmos endpoint, `todolist`, `documents`, and the two Google client ids.

AD-15’s module is “the plan, the Function App, the user-assigned identity, the function storage account, container `app-package`, the four storage data roles, and the Cosmos SQL role assignment.” The root owns the group, Cosmos, and the Static Web App, and outputs host names. Settings are not in that inventory. The Deferred pipeline “publishes the Function App zip and the Static Web App files.” It does not say it writes app settings. AD-14 also forbids Key Vault, so there is no third store.

The Terraform stack and the later pipeline can each assume the other wrote `GOOGLE_*` and `COSMOS_*`. One path can leave the names empty; another can set `COSMOS_DATABASE_NAME` to something other than `todolist` while AD-4 and AD-15 still create `todolist`. That is a real divergence between the API host and the clients that AD-3 binds to those audiences.

Fix in the spine: say the root stack (or `modules/function_host`) sets every AD-14 setting, that `COSMOS_DATABASE_NAME` and `COSMOS_CONTAINER_NAME` are `todolist` and `documents`, and that the Google client id values are Terraform inputs. Keep publishing the zip and the web files deferred.

`apiBaseUrl` is in better shape. AD-13 already fixes the value (production Function App host, or the local host on a developer machine). The injection mechanism can stay with the deferred pipeline.

**2. HTTP trigger auth is not anonymous-or-not.** Disposition: **autofix**.

AD-2 and AD-3 tell both clients to call with `Authorization: Bearer` only. AD-3 adds “A function key is not identity.” That stops a key from being used as `userId`. It does not say whether the HTTP triggers are anonymous, or whether a function key is still required to invoke them.

Flex Consumption functions still have an auth level. One unit can require a key (`?code=` or `x-functions-key`). The clients, as specified, will not send one. Web and Android then fail the same way, or one of them grows an undocumented key and the other does not.

Fix: state that every HTTP trigger is anonymous, that a function key is neither required nor accepted as a credential, and that the bearer Google ID token is the only credential.

**3. CORS allows origins and does not name headers.** Disposition: **autofix**.

AD-13: “CORS allows the Static Web App origin and the local Vite origin.” AD-6 requires `If-Match` on `PUT` and `DELETE`. AD-3 requires `Authorization`. The browser preflight will drop those headers unless the Function App allows them. The Android client does not send a preflight.

A CORS rule that allows the right origins and the wrong headers makes the web client fail etag writes while the phone succeeds. That is the cross-client divergence AD-2 and AD-6 exist to prevent.

Fix: allow `Authorization`, `Content-Type`, and `If-Match` (and expose `ETag` if any client reads it from the response rather than the JSON `etag`). Keep the two origins already named.

### Medium

**4. A drag that renumbers a day is not tied to the batch route.** Disposition: **autofix**.

AD-10: “A drag onto a day sets the due date and the dropped index, and that request renumbers the affected date.” AD-1 stores order only on each item’s `position`. The only multi-document write is `POST /items/changes` (AD-2), which rejects a missing etag and writes all or nothing.

The Rule never says the renumber request is that route. The web app can send the batch. The phone can `PUT` the dragged item and leave sibling positions stale. “Moving several items at once is web only” covers multi-select, not a one-item drop that must rewrite the date.

AD-7 lists `position` as a field and then constrains description, detail, state, and dates only. “`position` is a unique integer per due date” (AD-10) has no reject path. Sort’s final `id` tie-break hides duplicates instead of stopping them.

Fix: the renumber is one `POST /items/changes` of every item whose `position` or `dueDate` changed, each with its etag. The API rejects a batch that would leave two items on the same due date with the same `position`.

**5. The account document has no field list.** Disposition: **autofix**.

Item fields are closed (AD-7). The account document is “id `account`, type `account`,” created on first authenticated call (AD-3), with `schemaVersion` and partition `userId`. AD-3 forbids using email as the user id. It does not say whether email, name, or a picture is stored.

Two API authors can put different fields on `AccountDocumentV1`, and AD-5 will then freeze whichever shape landed first. Fix: name the account fields (likely `id`, `userId`, `type`, `schemaVersion` only) and say profile claims are not stored.

**6. Backup and alerting are silent.** Disposition: **discuss** (backup should be decided; alerting can be Deferred).

Operations is not an empty dimension. AD-14 decides the telemetry sink: Azure Monitor platform metrics on the Function App and Cosmos, no Application Insights, no Log Analytics workspace, no usage rows. The Deferred list repeats that prohibition.

What is missing is the rest of the envelope. Cosmos backup mode (periodic vs continuous, redundancy, retention) is not a rule, not Deferred, and not an open question. Platform default is periodic (two copies, no extra charge). Continuous 7-day backup storage is also free and changes restore. Two Terraform authors can pick either on a free-tier account. Alert rules and who looks at the platform metrics are likewise unstated. For this app, alerting can be an explicit Deferred item (“no alert rules in project 1”). Backup should be one sentence in AD-14 or AD-15, because it is account-level and painful to change later.

**7. Static Web App SKU and SPA fallback are unnamed.** Disposition: **autofix** the fallback; **discuss** the SKU.

AD-15’s root owns the Static Web App and does not name Free vs Standard. AD-13’s cost rule (no always-ready instances, Cosmos free tier) implies the free SWA SKU and never says it. A Standard SKU is a second bill shape.

The web app is a Vite client whose routes are not server routes (AD-2: the Static Web App serves the files only). Refresh on a client route needs a navigation fallback to `index.html`. That file is a deployment choice. Without it, one host config deep-links and another 404s.

**8. Idempotency keys are Deferred without a closed decision.** Disposition: **autofix** the Deferred line.

AD-2: “Create does not retry automatically.” The Deferred bullet is only “Client-generated idempotency keys for retried adds.” Unlike iOS and extra providers (“Project 1 does not include them”), this line does not forbid the feature. One client can add an idempotency header and the other, and the API, can ignore it. Change the bullet so project 1 has no idempotency key and no automatic retry of `POST /items`, matching AD-2.

### Low

**9. Vite is one patch behind.** Disposition: **autofix**. Stack pins Vite `8.3.5` (2026-10-21). Latest on the supported 8.3 line is `8.3.6` (2026-11-05). Same minor; pin the patch or say “8.3”.

**10. Node.js 24 is Maintenance LTS, not Active LTS.** Disposition: **discuss**. As of 2026-10-28, Node.js 26 is Active LTS (supported to 2029-04-30). Node.js 24 entered Maintenance on 2026-10-20 and is supported to 2028-04-30. Vitest 5.1.0 still accepts `^24`. The pin is a real, supported line. It is no longer the line Node recommends for new work. Either keep 24 on purpose or move the stack to 26.

**11. “xUnit 3” is the product line, not the package number.** Disposition: **ignore** or clarify. Current `xunit.v3` is 4.0.2 (2026-10-09), still branded Core Framework v3, and it supports `net10.0`. The stack label matches the product name. A reader can also install the 3.2.x packages. Optional clarity: “xUnit v3 (`xunit.v3` 4.x)”.

**12. Static Web Apps is “current React host.”** Disposition: **ignore**. The host is not a semver product. SKU is finding 7. The phrase itself is not a fake version.

**13. Cosmos emulator `vnext-latest` is a real floating tag.** Disposition: **ignore**. Microsoft documents `mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest` (GA). Dated tags also exist (`vnext-EN20261008`). The float matches “local running, not a test pyramid” (AD-11). Two laptops can pull different builds over time. Acceptable at this altitude.

**14. State-account bootstrap is unnamed.** Disposition: **defer**. AD-15 decides container `tfstate`, key `prod.tfstate`, versioning on, not committed, and not created by the root stack. Subscription, region, account name, and auth (access key vs Entra) are open. One Terraform repository means this is a single operator procedure, not two product units. Add one Deferred sentence so a later operator does not invent a second state account inside the production group (which would tangle AD-13).

**15. AD-11’s “infrastructure project” wording.** Disposition: **autofix** only if the Prevents line is touched. See the contradiction section. No rule conflict.

## Versions checked 2026-11-06

| Stack name | Spine | Check | Notes |
| --- | --- | --- | --- |
| .NET | 10 | Current LTS. Latest patch 10.0.14 (2026-10-25), supported to 2028-11-14. | .NET 11 is still release-candidate (GA dated 2026-11-10). Pinning 10 is correct today. |
| .NET MAUI | 10.0.120 | Latest stable patch (2026-10-21). Support to 2027-05-11. | 11.0.0 is prerelease only. |
| Azure Functions | Flex Consumption, isolated worker, .NET 10 | Matches the .NET 10 pin. | Not a separate version number. |
| React | 19.3.1 | Latest stable (2026-10-14). | |
| Vite | 8.3.5 | Real; one patch behind 8.3.6 (2026-11-05). Supported line is 8.3. | Finding 9. |
| TypeScript | 7.0.2 | Latest stable (npm `latest`, published 2026-07-08). 7.1 is dev/beta. | |
| Node.js | 24 | Maintenance LTS through 2028-04-30. Latest 24.22.2. Active LTS is 26. | Finding 10. |
| Vitest | 5 | Current major. Latest 5.1.0 (2026-11-02). Engines allow Node 24 and 26. | Major pin is enough. |
| xUnit | 3 (net10.0) | v3 line is current; package `xunit.v3` 4.0.2. | Finding 11. |
| Azure Cosmos DB | NoSQL, free tier (1000 RU/s, 25 GB) | Matches the current free-tier offer. | Backup mode still open (finding 6). |
| Cosmos emulator | vnext-latest | Documented GA image tag. | Finding 13. |
| Azure Static Web Apps | “current React host” | Host product is current; phrase is not a version. | Finding 12. |

## Dimensions

| Dimension | Where it stands |
| --- | --- |
| Client/server split, views, writes | Decided. AD-1, AD-2, AD-9, AD-10. |
| Identity | Decided, except trigger auth level (finding 2). |
| Data shape | Items decided. Account fields open (finding 5). |
| Consistency and errors | Decided. AD-6, AD-8. |
| Solution shape and tests | Decided. AD-11. |
| Presentation tokens | Decided. AD-12. Theme picker deferred. |
| Deployment | Decided. Terraform creates empty resources; publish is deferred. SPA fallback open (finding 7). |
| Environments | Decided. Production Azure plus local emulator. A second Azure environment is forbidden and deferred. |
| Infra / provider | Decided. One region, one production group, Flex Consumption, free-tier Cosmos, one Terraform stack. Setting values and SWA SKU open (findings 1 and 7). |
| Operations | Partial. Metrics decided. Backup and alerting silent (finding 6). |
| Release | Deferred, with the boundary stated (no Terraform publish). |

## Deferred check

Safe as written: iOS and extra providers and sharing and offline and admin; date-window queries (AD-1 already forces a full partition read); per-user week start (AD-9 locks Monday); phone theme picker; batch migration (AD-5 converts on read); second Azure environment and always-ready instances; Application Insights and the rest of that bullet; the publish pipeline, as long as finding 1 moves settings onto the Terraform stack.

Not safe until reworded: idempotency keys (finding 8).

## What this does not claim

Capability coverage against the PRD was not re-checked. The spine’s own map runs FR-1–FR-15 and NFR-1–NFR-3 and leaves no blank row. A requirement that exists only in the PRD would not show up here.
