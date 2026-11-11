# ToDoList project 1 — Architecture

The build contract for AD-1 through AD-15 is [ARCHITECTURE-SPINE.md](ARCHITECTURE-SPINE.md) in this folder. Those rule ids are stable. This document is the one to read for technologies, patterns, code quality, deployment, and the diagrams. The Terraform layout is in [terraform.md](terraform.md). AD-16 through AD-21 and the file map are still a draft in `1-3_Working/architecture-ToDoList-projects-2026-11-06/`. Until that draft is accepted, the Notes in [epics.md](epics.md) are the handoff for those rules.

Project 1 is a hobby app for a few people, built to a professional standard. The server stores facts. The web app and the Android app compute what those facts mean for today.

## Diagrams

| Diagram | File |
| --- | --- |
| Deployment and connectivity | [diagrams/deployment_connectivity.d2](diagrams/deployment_connectivity.d2) |
| Azure resources | [diagrams/deployment_azure-resources.d2](diagrams/deployment_azure-resources.d2) |
| App tier | [diagrams/structure_app-tier.d2](diagrams/structure_app-tier.d2) |
| Google sign-in | [diagrams/sequence_auth.d2](diagrams/sequence_auth.d2) |
| Documents | [diagrams/data_documents.d2](diagrams/data_documents.d2) |
| Save a new or existing item | [diagrams/sequence_save.d2](diagrams/sequence_save.d2) |
| Reorder | [diagrams/sequence_reorder.d2](diagrams/sequence_reorder.d2) |
| A save that loses the etag | [diagrams/sequence_etag-save.d2](diagrams/sequence_etag-save.d2) |

The D2 sources are the pictures. Render them with `d2` to SVG when you want an image.

## Deployment

Two places run the system. [diagrams/deployment_connectivity.d2](diagrams/deployment_connectivity.d2) is the client path. [diagrams/deployment_azure-resources.d2](diagrams/deployment_azure-resources.d2) is the Azure resource graph Terraform creates. [terraform.md](terraform.md) is the repository layout.

**Production** is one Azure region and one resource group. The region is a Terraform variable, and it has to be `westus2`, `centralus`, `eastus2`, `westeurope`, or `eastasia`. A Static Web App cannot be created outside that list. Cosmos stays in that same region.

- A Static Web App on the Free SKU serves the built React files and nothing else. It is not linked to the Function App.
- A Function App on the Flex Consumption plan runs the HTTP API. It uses the .NET 10 isolated worker, 2048 MB instances, and no always-ready instances, so it sleeps when nobody is calling it. HTTP triggers are anonymous and the route prefix is empty, so clients call `/items` on that host with the Google bearer token and no function key. Flex Consumption runs a package published through the function deployment endpoint into container `app-package`. Copying a zip into the container does not deploy it.
- A storage account (StorageV2, Standard LRS, hot, TLS 1.2) holds that zip in blob container `app-package` and the host data the Functions runtime needs. Blob public access is off. Shared keys are off. It does not store to-do items.
- One user-assigned managed identity is attached to the Function App. On the storage account it has Storage Blob Data Owner, Storage Blob Data Contributor, Storage Queue Data Contributor, and Storage Table Data Contributor. Those four are what the current Microsoft Flex Consumption identity template assigns when shared keys are disabled; the host uses blobs, queues, and tables even though the API is HTTP only. On the Cosmos account the same identity has the Cosmos SQL role Cosmos DB Built-in Data Contributor. An Azure RBAC role on the account does not grant item reads and writes.
- A Cosmos DB account, API for NoSQL, free tier (the allowance is 1,000 RU/s and 25 GB, chosen when the account is created). There is one free-tier account per subscription. This one is production only. Container `documents` is provisioned at 1,000 RU/s so the allowance covers it. Consistency is Strong, so a read on a new instance sees a write that just committed. The Function App does not keep a Cosmos session token. Account keys are disabled. The network is public, with no private endpoint, because the Function App has no virtual network and its outbound addresses change. Database `todolist`, container `documents`, partition key `/userId`. Backup stays the account default.

There is no Key Vault, no virtual network, no Application Insights resource, no Log Analytics workspace, and no alert rules. Usage and volume are the Azure Monitor platform metrics on the Function App (request count, HTTP 5xx, on-demand execution count and execution units) and on Cosmos (requests and request units). Ingestion of those metrics is free. Microsoft’s metrics article retains them 93 days. The portal cost view shows the dollars.

The Function App settings are:

| Setting | Value |
| --- | --- |
| `AzureWebJobsStorage__accountName` | function storage account name |
| `AzureWebJobsStorage__credential` | `managedidentity` |
| `AzureWebJobsStorage__clientId` | the identity's client id |
| `AZURE_CLIENT_ID` | the same client id, so the Cosmos SDK's `DefaultAzureCredential` selects this identity |
| `COSMOS_ACCOUNT_ENDPOINT` | the Cosmos account endpoint |
| `COSMOS_DATABASE_NAME` | `todolist` |
| `COSMOS_CONTAINER_NAME` | `documents` |
| `GOOGLE_WEB_CLIENT_ID` | web audience |
| `GOOGLE_ANDROID_CLIENT_ID` | Android audience for the Play Store package |

No Cosmos key, no storage connection string, and no Application Insights key. The Google client ids are not secrets. CORS allows the Static Web App's default hostname and the local Vite origin (`http://localhost:5173` unless the web project pins another port).

**Local** work runs Vite, the same Function App, and the Cosmos vNext emulator (`mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest`) in Docker, started with HTTPS. The .NET SDK needs that HTTPS endpoint. Stop the container when you are done. It does not bill Azure, and it does not hold production items. The emulator has no Entra login, so local settings hold the emulator connection string and the API uses that instead of the managed identity. Those local settings are not in Terraform.

The browser loads the site from the Static Web App, then calls the Function App URL in `apiBaseUrl`. The Android app, shipped through the Play Store, calls that same kind of setting. Neither client has a connection string.

Terraform creates this resource group and outputs the two host names. Publishing the Function App zip and the Static Web App files is the application repository's job, and that pipeline is still later.

## Stack

| Piece | Choice |
| --- | --- |
| API | C# on .NET 10, Azure Functions isolated worker with ASP.NET Core integration, Flex Consumption. Tests use xUnit 3 on net10.0. |
| Android | .NET MAUI 10.0.120, published to the Play Store. iOS is later. |
| Web | Vite 8.3.5, React 19.3.1, TypeScript 7.0.2, Node.js 24. Tests use Vitest 5. |
| Data | Cosmos DB, API for NoSQL, free tier in production, vNext emulator locally |
| Sign-in | Google ID tokens. Web client id and Android client id. User id is the Google `sub`. |

Versions were checked on 2026-11-04 against Microsoft's current docs. MAUI 10 support ends 2027-05-11. .NET 10 support ends 2028-11-14. The Azure resource contract, including Flex Consumption with a user-assigned identity and .NET 10, was checked again on 2026-11-06.

## App tier

| Project | Responsibility |
| --- | --- |
| `web` | React UI. Checks fields before send. Computes views. Theme lives in `tokens.css`. |
| `ToDoList.App` | MAUI UI. Opens on Add new. Calls Core for field checks and Scheduling for views. |
| `ToDoList.Scheduling` | Today, week, and order. Referenced by MAUI and by xUnit. |
| `ToDoList.Core` | Item rules and schema converters. No Azure types. The API and MAUI both reference it. |
| `ToDoList.Api` | HTTP edge. Checks the Google token, checks the field contract again, talks to Cosmos. |
| `ToDoList.Tests` | xUnit for Core and Scheduling. |
| `contracts/item-fields.json` | The only copy of the field limits. |
| `contracts/view-examples.json` | Shared cases. xUnit and Vitest both run them. |

The web app does not reference the C# projects. Cosmos is a concrete class inside the API. There is no storage interface and no extra infrastructure project.

## Data

One database named `todolist`, one container named `documents`. The partition key is `userId`, the Google `sub`.

The account document id is `account`. It holds the person and, later, account settings. Each to-do item is its own document, including completed ones, so history does not accumulate inside one document. The API assigns the item id. `GET /items` returns a JSON array of every item in the partition. The `etag` string is the Cosmos `_etag`, including any quotes, and `If-Match` sends that string back unchanged. The Function App does not cache a person's items, so that read, including the reload after a lost etag, comes from Cosmos. The client decides what is Today, what is open, and what is hidden. `POST /items` returns 201 with the new id and etag. `PUT` and `DELETE` send `If-Match`. A reorder or a web multi-move is `POST /items/changes`, and it writes every listed item or none. `userId` comes from the Google token. The server sets `id`, `userId`, and `schemaVersion`.

Every document carries `schemaVersion` (an integer, starting at 1) and Cosmos `_etag`. Account versions and item versions are separate sequences. Storage classes are named `AccountDocumentV1` and `ItemDocumentV1`. Those names stay when a v2 appears. The rest of the code uses `Account` and `Item`, which always mean the current fields. A read deserializes the stored class and converts it. A save writes the current class. A version the code does not know fails the read. A new required field gets its default in the converter.

`_etag` is not the schema version. Cosmos changes it on every successful save. Updates and deletes send it back. If another client saved first, the API returns 409 and this client snaps back and reloads. There is no merge.

## Behavior the two clients share

- Dates are calendar dates, `YYYY-MM-DD`, with no time zone. Today is the device's local calendar date, not UTC. The app recomputes Today when it returns to the foreground. Today shows open items due on or before that date, and done items due that date. Sort is due date, then position, then id.
- The week starts Monday, from one config value. Next week is that Monday. Next month is the first.
- Description is required, trimmed, 1 to 200 characters. Detail is optional, up to 2,000. State is `not-started`, `in-progress`, or `done`.
- Setting done stores the completed date the client sends, which is that device's local calendar date, and leaves the due date alone. The server does not replace it with its own clock. Leaving done clears the completed date.
- A reschedule action places the item at the end of the target date. A drag onto a day lands at the drop index, and that save renumbers the day.
- Hide completed is stored on the device. It does not call the API.
- Moving several items at once is web only. The web sends `POST /items/changes` with every changed item. The API writes that batch, or writes none of it.
- Signed-out web shows sign-in before the list. Signed-out Android can hold a description and then ask to sign in. Each client refreshes its Google token quietly while it is in use. Signing out of one does not sign out the other.

## Errors

The API returns problem-details.

| Status | Client |
| --- | --- |
| 400 | Shows the field messages. The same sentences the form uses. |
| 401 | Starts sign-in on that client. |
| 409 | Snaps back and reloads. |
| 5xx, or no response | Shows `Error reaching server`. Typed text stays. A failed move snaps back. |

The body does not include exception text or Cosmos messages.

## Theme

`tokens.css` defines `--color-background`, `--color-surface`, `--color-line`, `--color-text`, `--color-text-muted`, `--color-accent`, `--color-surface-picked`, and `--color-error`. Component classes use those variables and do not contain hex values. A later theme is another sheet, selected with `data-theme` on the page. Project 1 ships the dark sheet. MAUI uses the same token names in a resource dictionary and ships dark only.

## Rules for implementation

These are here so a later agent does not "improve" the design into a larger one.

- Do not add an interface unless two implementations exist.
- Do not add a project beyond the table above.
- Check a request at the client, and once more at the API. Do not check it again inside Core after the API has accepted it.
- Keep a behavior in one readable type. Do not split it across a file per method.
- Do not return internal errors to the client.
- Do not point a local run at the production Cosmos account.
- Do not put a Cosmos key or a storage connection string on the Function App.
- Do not store lists, and do not compute Today on the server.
- Do not cache a person's items in the Function App. A reload reads Cosmos.

## Left for later

iOS, other sign-in providers, sharing, offline, admin tools, paging, per-user week start, a phone theme picker, a background migration of old documents, idempotency keys for a retried add, and the pipeline that publishes the Function App zip and the Static Web App files. None of these are required to describe the Azure resources. The publish pipeline is still required before the app is released.
