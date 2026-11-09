# Version reality-check — 2026-11-06 additions

Reviewed: `ARCHITECTURE-SPINE.md` (final, updated 2026-11-06), the deployment section of `1-4_OutputDocs/architecture.md`, and `1-4_OutputDocs/terraform.md`.
Evidence of prior research: `.memlog.md` lines for 2026-11-04 and 2026-11-06. The spine records names only.
Checked against Microsoft Learn, the Terraform `hashicorp/azurerm` registry (`latest`), and `Azure/azure-quickstart-templates` `function-app-flex-managed-identities/main.bicep` on `master`, on 2026-11-06. Nothing was applied in Azure. The spine and the two output docs were not modified.

## Verdict

The 2026-11-06 hosting claims were checked against real sources, and the load-bearing ones still match: .NET 10 isolated on Flex Consumption, the four storage data-role ids, Cosmos DB Built-in Data Contributor versus Azure RBAC, `local_authentication_enabled = false`, `free_tier_enabled`, the Static Web Apps Free SKU, and free platform-metric ingestion. They are not bare assertions.

Three written Terraform details would fail or stall an apply if copied as names: the storage public-access argument, the Cosmos SQL role-assignment id, and one region shared with Static Web Apps. Platform-metric retention is three days off the metrics article. `free_tier_enabled` does not itself provision 1,000 RU/s.

## What was re-fetched

| Claim | Result on 2026-11-06 | Source |
| --- | --- | --- |
| Flex Consumption with shared keys off uses one user-assigned identity and Storage Blob Data Owner, Blob Data Contributor, Queue Data Contributor, and Table Data Contributor. | Holds. `main.bicep` on `master` still assigns those four at storage-account scope, principal type ServicePrincipal, with the same ids: `b7e6dc6d-f1e8-4753-8033-0f276bb0955b`, `ba92f5b4-2d11-453d-a403-e96b0029c9fe`, `974c5e8b-45b9-4653-ba55-5f855dd0fb88`, `0a9a7e1f-b9d0-4cc4-a60d-0319b160aaa3`. Shared key access is `allowSharedKeyAccess = false`. Host settings are still `AzureWebJobsStorage__accountName`, `__credential` = `managedidentity`, `__clientId`. | [Identity Bicep](https://github.com/Azure/azure-quickstart-templates/blob/master/quickstarts/microsoft.web/function-app-flex-managed-identities/main.bicep) |
| Runtime list includes `dotnet-isolated` and version `10`. Deployment container is required. | Holds, with a stale default. Allowed runtime names include `dotnet-isolated`. Allowed versions include `10` (not `10.0`). The parameter default is still `8.0`. The deployment container is required; this file names it `app-package-${appName}-${token}`, not the literal `app-package`. Plan SKU is `FC1`. Instance memory allowed in this file is 2048 or 4096, default 2048. Always-ready is omitted. The file also creates Application Insights and assigns Monitoring Metrics Publisher `3913510d-42f4-4e42-8a64-420c390055eb`. | Same Bicep. [Flex plan](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan) lists C# isolated .NET 8, 9, and 10. |
| .NET 10 isolated is the Flex path. In-process ends 2026-11-10. .NET 10 is not on Linux Consumption. | Holds on the current language pages. Flex Consumption is the recommended serverless plan and lists .NET 10 isolated. Isolated worker support table: .NET 10 GA through 2028-11-14. .NET 10 cannot run on Linux Consumption. In-process support ends 2026-11-10 and stops at .NET 8. Minimum packages unchanged: `Microsoft.Azure.Functions.Worker` 2.50.0+ and `Azure.Functions.Sdk` 1.0.0+. | [Flex plan](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan), [supported languages](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages), [isolated worker guide](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide) |
| `local_authentication_enabled = false` disables Cosmos keys. `free_tier_enabled` still exists. | Holds on the current registry page. `local_authentication_enabled` defaults to `true`. `free_tier_enabled` defaults to `false` and forces a new account when changed. The 4.x provider still has the older `local_authentication_disabled` until 5.0; that name is the opposite boolean and conflicts with the new one. | [azurerm_cosmosdb_account](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/cosmosdb_account) |
| Data-plane access is Cosmos DB Built-in Data Contributor, id `00000000-0000-0000-0000-000000000002`. An Azure RBAC role on the account does not grant item reads and writes. | Holds. The id and the included `dataActions` are unchanged. Learn states these roles are distinct from Azure RBAC, and that Owner and Contributor do not provide Entra data-plane access. | [Data-plane security](https://learn.microsoft.com/en-us/azure/cosmos-db/reference-data-plane-security), [Entra data access](https://learn.microsoft.com/en-us/azure/cosmos-db/how-to-connect-role-based-access-control) |
| Static Web Apps Free SKU still exists. | Holds. Learn plans are Free and Standard. Dedicated is retired as of 2025-10-31. `azurerm_static_web_app` `sku_tier` / `sku_size` are `Free` or `Standard`, default `Free`. Free has no SLA, 100 GB bandwidth with no overage, and 250 MB per environment. | [Plans](https://learn.microsoft.com/en-us/azure/static-web-apps/plans), [quotas](https://learn.microsoft.com/en-us/azure/static-web-apps/quotas), [azurerm_static_web_app](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/static_web_app) |
| Platform metrics are free. | Ingestion holds. Standard platform metrics have no ingestion or storage charge. Advanced platform metrics are a separate paid opt-in. Named signals exist as platform metrics: Function App `Requests`, `Http5xx`, `OnDemandFunctionExecutionCount`, `OnDemandFunctionExecutionUnits`; Cosmos `TotalRequests`, `TotalRequestUnits`. | [Metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/metrics/data-platform-metrics), [cost](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/cost-usage), [Functions reference](https://learn.microsoft.com/en-us/azure/azure-functions/monitor-functions-reference), [Cosmos reference](https://learn.microsoft.com/en-us/azure/cosmos-db/monitor-reference) |
| Platform metrics are retained 90 days. | Off by the metrics article. That article says platform and custom metrics are stored 93 days. The pricing page says both “retained for 90 days” and, in the FAQ, 93 days. | [Metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/metrics/data-platform-metrics), [pricing](https://azure.microsoft.com/en-us/pricing/details/monitor/) |
| Analytics Logs include 5 GB/month, then about $2.30/GB in East US. | The 5 GB grant still holds (memlog only; not a spine rule). The pricing page still says the first 5 GB/month per billing account of Analytics Logs are free. The fetched page renders the per-GB rate as a placeholder, so $2.30 was not reconfirmed. | [Monitor pricing](https://azure.microsoft.com/en-us/pricing/details/monitor/) |

Related fit that the same pages still support:

- Instance memory 2048 MB is a supported Flex size and the documented default for most apps. Allowed sizes on the plan page are 512, 2048, and 4096. `azurerm_function_app_flex_consumption` `instance_memory_in_mb` defaults to 2048.
- Always-ready defaults to 0 when the block is omitted. On-demand Flex billing has a monthly grant of 250,000 executions and 100,000 GB-s per subscription. Always-ready has no free grant. AD-13 and AD-14 match the omit-the-block case.
- `AZURE_CLIENT_ID` is still the environment variable `DefaultAzureCredential` uses to select a user-assigned identity. The .NET Identity readme documents that.
- `azurerm_function_app_flex_consumption` accepts `runtime_name` `dotnet-isolated`, `storage_authentication_type` `UserAssignedIdentity`, and requires `storage_user_assigned_identity_id` in that case. `https_only` defaults to false, so the guide is right to set it. `shared_access_key_enabled` defaults to true, so the guide is right to set it false. `maximum_instance_count` is optional (1–1000); omitting it is a real “platform default.”
- Plan SKU `FC1` is the Flex Consumption SKU on `azurerm_service_plan`.

## Findings

### 1. The storage public-access argument in the Terraform guide is the pre-3.0 name

- **Location:** `terraform.md`, Function host module, storage account: `allow_blob_public_access` false.
- **Trigger:** Current `azurerm_storage_account` documents `allow_nested_items_to_be_public`. The 3.0 upgrade guide removed `allow_blob_public_access`. The Azure API name in the identity Bicep is still `allowBlobPublicAccess`. The guide mixed that API name into a Terraform argument list that otherwise uses current provider names (`shared_access_key_enabled`, `local_authentication_enabled`).
- **Guard:** Write `allow_nested_items_to_be_public = false`. That is already the provider default.
- **Consequence:** An apply that copies the guide’s argument name fails validation on the current provider. The intended setting (no public blob opt-in) is the default, so the failure is the name, not the security choice.

### 2. The Cosmos SQL role id is right; the Terraform argument is not that bare GUID

- **Location:** `terraform.md`, “definition id `00000000-0000-0000-0000-000000000002`.” AD-14 names the role and does not show the id.
- **Trigger:** The GUID is still the built-in role’s name. `azurerm_cosmosdb_sql_role_assignment.role_definition_id` is “the resource ID of the Cosmos DB SQL Role Definition.” The data source `azurerm_cosmosdb_sql_role_definition` is what takes the GUID and returns `.id`. The registry example assigns that `.id`. The same shape applies to the four storage roles: the GUIDs match the Bicep, and `azurerm_role_assignment` wants `role_definition_name` or a full role-definition resource id.
- **Guard:** Look the built-in definition up with the GUID, and assign `role_definition_id` from the data source `.id` (account id plus `/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002`). Prefer `role_definition_name` for the four storage roles.
- **Consequence:** Pasting the GUID into `role_definition_id` does not match the current argument contract. The role choice itself is still the one that grants item reads and writes. An Azure control-plane role still does not.

### 3. One region for every resource includes Static Web Apps, which still reject most regions

- **Location:** AD-14 and AD-15 (“one Azure region”). `terraform.md`: one `location` variable for every regional resource and the Cosmos geo location. Deployment section: one region, chosen at first apply.
- **Trigger:** No `(version)` line checks where a Static Web App can be created. The resource type `Microsoft.Web/staticSites` is still limited. Microsoft’s 2026-03 region note and the longstanding service error list the same five: `westus2`, `centralus`, `eastus2`, `westeurope`, `eastasia`. `eastus` is called out as a failure. The memlog’s East US log price is a different region from those five. Static assets are global; the location is where the resource and its managed Functions host are created. This stack does not use that managed host, and the region limit still applies to the resource.
- **Guard:** Constrain `location` to a region that can create `Microsoft.Web/staticSites`, or split the Static Web App location from the Cosmos and Functions location. Do not treat “any region” as safe.
- **Consequence:** The first apply in a normal region such as East US creates the group, Cosmos, and the function host, then fails on the Static Web App. That is the production resource group AD-13 describes.

### 4. Free tier is a discount flag, not the throughput setting

- **Location:** AD-13, AD-14, stack table “1000 RU/s, 25 GB.” `terraform.md`: `free_tier_enabled` only. No database or container throughput.
- **Trigger:** `free_tier_enabled` is optional, default false, and forces a new account if changed. It does not set RU/s. A provisioned NoSQL account (this one is not serverless; free tier is not available on serverless) still needs throughput on the database or the container. The discount then covers the first 1,000 RU/s and 25 GB on that one account per subscription. The 2026-11-04 free-tier check established the allowance. This pass confirmed the Terraform flag still exists and still does not replace throughput.
- **Guard:** Set throughput to 1000 on `todolist` or on `documents`, and set `free_tier_enabled = true` on the account. One such account per subscription.
- **Consequence:** An apply that only flips the flag either fails container creation or provisions a throughput the free allowance does not match. The 1,000 RU/s figure in the stack table is the allowance, not an argument the guide names.

### 5. “Kept 90 days” matches the pricing banner, not the metrics article

- **Location:** AD-14. `architecture.md` deployment: free platform metrics “kept 90 days.” Memlog 2026-11-06 says the same.
- **Trigger:** Ingestion of standard platform metrics is free, and the named Function App and Cosmos metrics are standard metrics. The metrics article stores them 93 days. The pricing FAQ also says 93 days, after a banner that says 90. Metric REST API queries are not unlimited: the first 1,000,000 calls per month are free, then billed. Portal charts are the path this design uses. Advanced platform metrics are paid and are not these signals.
- **Guard:** Say ingestion is free and retention follows Azure Monitor platform metrics (the article says 93 days). Do not add a diagnostic setting; that is the paid log path the decision already excludes.
- **Consequence:** A reader who treats 90 as a hard purge date is early by a few days. A reader who polls the metrics API at high volume can still get a bill. The decision to skip Log Analytics is still the free path.

## Checked and still a fit

| Decision | Check on 2026-11-06 |
| --- | --- |
| Four host-storage roles, even for an HTTP-only API | The identity template still assigns blob, queue, and table roles when shared keys are off. Flex Consumption how-to, separately, says the deployment identity needs Storage Blob Data Contributor on the deployment account. Blob Data Owner is redundant with Blob Data Contributor; the template still assigns both. Matching the template is what AD-14 says. |
| Literal container name `app-package` | The platform container name is chosen by the app. The template only uses `app-package-` as a prefix plus a unique suffix. A container named `app-package` is valid. It is not what copying the template emits. |
| Package / zip deployment is the Flex code path | Flex runs from a package in the deployment container. The package is a zip. Publishing tools push it to the app’s deployment endpoint. Putting a zip in the container by hand does not deploy it. Do not set `WEBSITE_RUN_FROM_PACKAGE`. “The deployment zip is the only code-deployment method” is the right family of deployment. The container is not a drop folder. |
| No Application Insights settings | Correct for AD-14. The identity template’s extra role and settings (`APPINSIGHTS_INSTRUMENTATIONKEY`, `APPLICATIONINSIGHTS_AUTHENTICATION_STRING`, Monitoring Metrics Publisher) are the part the guide tells you to leave out. |
| Owner for apply, Contributor cannot assign roles | Owner can create the resource group, the four Azure RBAC storage assignments, and the Cosmos SQL role assignment. Contributor cannot create `Microsoft.Authorization` role assignments, so it cannot grant the storage data roles. Cosmos SQL assignments are DocumentDB control-plane writes, which Contributor can perform. The guide’s sentence is about role assignment in general and is the right bar for this stack. |
| `local_authentication_enabled = false` | Matches current registry docs. On 4.x, do not also set `local_authentication_disabled`. That older argument is removed in provider 5.0. |
| Keys off and the portal | Disabling local auth is what the provider describes: only managed identity and Entra. Tools that speak only account keys stop working. Data Explorer needs an Entra identity that holds a data-plane role. |
| Scale-out cap left unset | `maximum_instance_count` is optional. The identity Bicep instead defaults it to 100 and rejects values under 40. “Platform default” means omit the property, not copy that parameter. |
| CORS and `http://localhost:5173` | Not a versioned Azure fact. Vite’s default port is still 5173. Unchanged from the earlier stack check. |

## Earlier technology decisions, this pass

AD-1 through AD-12 are product rules. This pass did not re-litigate them. The technology pins they depend on were reality-checked on 2026-11-04 in `reviews/review-versions.md`. Today’s language-page fetch again supports the .NET 10 and Flex half of that review: LTS through 2028-11-14, .NET 8 and .NET 9 end 2026-11-10, in-process ends the same day.

Not re-fetched today: npm pins (React 19.3.1, Vite 8.3.5, TypeScript 7.0.2), Node.js 24, Vitest 5, xUnit 3, MAUI 10.0.120, and the Cosmos vNext emulator tag. Those rows now exist in the stack table. The memlog still has no `(version)` line that names them. The 2026-11-04 review is the check. MAUI’s next chance to move is the 2026-11-10 patch day, which has not arrived.

`architecture.md` still says “Versions were checked on 2026-11-04.” The deployment section was written on 2026-11-06. The 2026-11-06 checks live in the memlog, not in that sentence.

## Out of date in the sources, not in the decision

- The identity Bicep defaults `functionAppRuntimeVersion` to `8.0`. .NET 8 support ends 2026-11-10. Version `10` is allowed; it is not the default. The allowed list is also behind the Flex language table (no Node.js 22 or 24).
- [Functions versions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-versions) still has a paragraph that calls `net10.0` preview and lists `net9.0`, `net8.0`, and `net48` as the isolated choices. The Flex plan page and the supported-languages table say .NET 10 is GA through 2028-11-14. Follow those two, and set runtime version `10`, not `10.0`, which is what this Bicep allow-list and the current Functions guidance use.
- Azure Monitor’s own pages disagree on 90 versus 93 days. The metrics article is the retention statement.
- The monitor how-to for request units still says the portal chart can show at most the past 30 days. That sentence disagrees with the 93-day retention article.
