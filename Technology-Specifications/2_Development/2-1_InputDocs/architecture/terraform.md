# ToDoList Terraform

This is the layout for the repository that creates the production Azure resources in [architecture.md](architecture.md). The picture is [diagrams/deployment_azure-resources.d2](diagrams/deployment_azure-resources.d2). The rules are AD-14 and AD-15 in the spine.

One repository. One production stack. Local development stays the Vite app, the Function App, and the Cosmos emulator. It is not a Terraform environment.

Terraform creates empty resources and outputs host names. It does not contain the React app or the .NET projects, and it does not publish the zip or the static files. That publish pipeline is later.

## Who can apply

The account that runs Terraform must be allowed to create resource groups and to create role assignments. Owner on the subscription covers that, including the Cosmos SQL role assignment. Contributor on its own cannot assign roles.

Cosmos free tier allows one account per subscription. This stack is that account.

## State

State is Terraform's record of the resource ids it created. The to-do documents are in Cosmos either way. A lost state file leaves the resources in place. The next apply tries to create them again, Azure rejects the duplicate names, and the apply stops. Rebuilding the file is `terraform import`, once per resource. Restoring a versioned blob is the shorter path.

The state blob lives in its own resource group and storage account, created by `bootstrap/` before the root stack exists. The root stack does not manage that account, so replacing the function host cannot delete the state. Blob versioning is on. The container is `tfstate`. The blob key is `prod.tfstate`. The root backend uses Azure AD (`use_azuread_auth`), so the backend block holds names only and can be committed. State files stay out of git.

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-todolist-tfstate"
    storage_account_name = "sttodolisttfstate"
    container_name       = "tfstate"
    key                  = "prod.tfstate"
    use_azuread_auth     = true
  }
}
```

`sttodolisttfstate` has to be globally unique. If that name is taken, change it in bootstrap and in this block together. Bootstrap itself uses the local backend. Its state file is gitignored. Losing it means importing the state storage account, which is one resource group and one account.

## Repository

```text
bootstrap/
  main.tf            # state resource group, storage account, container, versioning
  variables.tf
  outputs.tf
  versions.tf
modules/
  function_host/
    main.tf
    variables.tf
    outputs.tf
main.tf              # production resource group, Cosmos, Static Web App, module call
variables.tf
outputs.tf
providers.tf
versions.tf
backend.tf
prod.tfvars.example
.gitignore           # .terraform/, *.tfstate, *.tfstate.*, crash.log, override files
```

`modules/function_host` is the only module. It is the set of resources that have to stay wired to one identity. Everything that exists once, and is not part of that wiring, stays in the root.

| Path | Owns |
| --- | --- |
| `bootstrap/` | State resource group, state storage account (Standard LRS, versioning on), container `tfstate` |
| root | Production resource group, Cosmos account, database `todolist`, container `documents`, Static Web App |
| `modules/function_host` | User-assigned identity, function storage account, container `app-package`, Flex Consumption plan, Function App, four storage data roles, Cosmos SQL role assignment |

The root passes the module the Cosmos account id (for the role assignment), the Cosmos endpoint URL (for `COSMOS_ACCOUNT_ENDPOINT`), the database name, the container name, both Google client ids, and the Static Web App hostname. The module writes every AD-14 app setting. The storage role assignments are scoped to the function storage account. The Cosmos role assignment uses the account id, not the endpoint URL.

No module for the Static Web App, Cosmos, naming, or a local Azure environment.

## Names

One `location` variable. Every regional resource and the Cosmos geo location use it. Allowed values are `westus2`, `centralus`, `eastus2`, `westeurope`, and `eastasia`. `prod.tfvars.example` uses `eastus2`.

One `name_prefix` (default `todolist`) and one short `name_suffix` for the globally unique names. Storage account names are lowercase letters and digits, 3 to 24 characters.

| Resource | Name |
| --- | --- |
| State resource group | `rg-todolist-tfstate` |
| State storage account | `sttodolisttfstate` plus a suffix if the name is taken |
| Production resource group | `rg-todolist-prod` |
| User-assigned identity | `id-todolist-prod` |
| Function storage account | `sttodolistfunc` plus the suffix |
| Deployment container | `app-package` |
| Plan | `plan-todolist-prod` |
| Function App | `func-todolist-prod` plus the suffix |
| Cosmos account | `cosmos-todolist-prod` plus the suffix |
| Static Web App | `swa-todolist-prod` |
| Database | `todolist` |
| Container | `documents` |

`prod.tfvars.example` lists `location`, `name_prefix`, `name_suffix`, `GOOGLE_WEB_CLIENT_ID`, and `GOOGLE_ANDROID_CLIENT_ID`. Those client ids are public (they ship in the clients). Copy the example to `prod.tfvars` for apply. Commit the example.

## Function host module

Match the current Microsoft Flex Consumption identity template, without its Application Insights resources. Runtime `dotnet-isolated`, version `10`. Plan SKU `FC1`. `https_only`. Minimum TLS 1.2. Instance memory 2048 MB. No always-ready instances. Leave the scale-out cap at the platform default.

Storage account: kind `StorageV2`, SKU `Standard_LRS`, access tier Hot, `allow_nested_items_to_be_public` false, `shared_access_key_enabled` false, minimum TLS 1.2, public network access on, default network action Allow. Container `app-package`, public access off. Assign the four storage roles with `role_definition_name`, not a bare GUID.

Identity type on the Function App is user-assigned, and that identity is the deployment identity for the blob container.

Role assignments on the function storage account, principal type ServicePrincipal:

| Role | Definition id |
| --- | --- |
| Storage Blob Data Owner | `b7e6dc6d-f1e8-4753-8033-0f276bb0955b` |
| Storage Blob Data Contributor | `ba92f5b4-2d11-453d-a403-e96b0029c9fe` |
| Storage Queue Data Contributor | `974c5e8b-45b9-4653-ba55-5f855dd0fb88` |
| Storage Table Data Contributor | `0a9a7e1f-b9d0-4cc4-a60d-0319b160aaa3` |

App settings are the table in [architecture.md](architecture.md). Do not set `APPINSIGHTS_INSTRUMENTATIONKEY` or `APPLICATIONINSIGHTS_CONNECTION_STRING`.

CORS allowed origins: the Static Web App default hostname (with `https://`) and `http://localhost:5173`. Functions CORS is an origin list. The browser preflight for `Authorization` and `If-Match` has to succeed for those origins. Local `host.json` sets the HTTP route prefix to an empty string, and each HTTP trigger is anonymous. Terraform does not publish that file.

## Root resources

Cosmos account: kind GlobalDocumentDB, `free_tier_enabled` true, Strong consistency, one geo location equal to `location`, keys disabled, public network access, no virtual network filter and no private endpoint. On the current `azurerm` provider, keys are disabled with `local_authentication_enabled = false`. Database `todolist`. Container `documents`, partition key `/userId`, throughput 1000. The free-tier flag does not set that throughput. It discounts the first 1,000 RU/s on this one account. Leave backup at the provider default.

The Cosmos SQL role assignment is created in the module, because it belongs to the identity. Built-in role Cosmos DB Built-in Data Contributor. Its name GUID is `00000000-0000-0000-0000-000000000002`. `azurerm_cosmosdb_sql_role_assignment.role_definition_id` needs the role's resource id, which is the account id plus `/sqlRoleDefinitions/00000000-0000-0000-0000-000000000002`. Read that id from `azurerm_cosmosdb_sql_role_definition`. Do not pass the bare GUID. Do not invent a custom role, and do not assign an Azure control-plane role in its place. The assignment can take a few minutes to take effect after apply.

Static Web App: SKU Free. No linked backend.

## Outputs

| Output | Consumer |
| --- | --- |
| Function App default hostname | `apiBaseUrl` in the web build and the Android app |
| Static Web App default hostname | the site URL, and the Function App CORS origin |
| Cosmos endpoint | already written to `COSMOS_ACCOUNT_ENDPOINT` |
| Static Web App deployment token | sensitive. The later publish pipeline uses it. It is not a Function App setting. |

## Apply order

1. `terraform init` and `terraform apply` in `bootstrap/`.
2. Put the resulting storage account name in `backend.tf` if it differs from the example.
3. `terraform init` and `terraform apply -var-file=prod.tfvars` in the root.

Use the current stable Terraform and the current `hashicorp/azurerm` provider. Pin both in `versions.tf` to the versions you applied with.

## Left out

A second Azure environment. Application Insights and Log Analytics. A virtual network or private endpoints. Key Vault. Publishing the zip or the static files. Modules for resources this stack creates only once.
