# Version reality-check — Architecture Spine

Reviewed: `ARCHITECTURE-SPINE.md` (draft, updated 2026-11-04).
Evidence of prior research: the run memlog (`.memlog.md`), not the spine. The spine records names and versions only.
Checked against the public web on 2026-11-04. This workspace has no application repository, so nothing was compared to an existing project or a generated starter on disk. Starter defaults were read from the current `create-vite` template and from current vendor docs.
The spine was not modified.

## Verdict

The six version claims written in the coaching log on 2026-11-04 are still true. The spine is not fully reality-checked. React 19, Vite 8, Node.js 22, TypeScript, Vitest, and xUnit are named or pinned with no version entry in that log. The live Vite React TypeScript starter, which the memlog says the web client is, disagrees with the Node pin and uses TypeScript 7, which the stack table does not name.

## What the coaching log actually checked

These are the only `(version)` lines in `.memlog.md`. Each was re-fetched today.

| Claim in the log | Result on 2026-11-04 | Source |
| --- | --- | --- |
| .NET 10 is LTS through 2028-11-14. .NET 8 and .NET 9 support ends 2026-11-10. | Holds. Support-policy table: .NET 10 LTS, active, end of support 2028-11-14. .NET 8 and .NET 9 end 2026-11-10. Latest patch is 10.0.14 (2026-10-25). The lifecycle page lists the same end as 2026-11-15 06:59:59 PT, which is the end of 2026-11-14 in Redmond. | [Support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core), [downloads](https://dotnet.microsoft.com/en-us/download/dotnet), [.NET 10 release notes](https://github.com/dotnet/core/blob/main/release-notes/10.0/README.md) |
| Flex Consumption isolated worker supports .NET 10. In-process support ends 2026-11-10. .NET 10 is not on Linux Consumption. | Holds. Functions 4.x isolated worker lists .NET 10 as GA through 2028-11-14. Docs say .NET 10 cannot run on Linux Consumption and to use Flex Consumption. In-process support ends 2026-11-10 and stops at .NET 8. Minimum packages for .NET 10: `Microsoft.Azure.Functions.Worker` 2.50.0+ and `Azure.Functions.Sdk` 1.0.0+. The current sample project SDK is `Azure.Functions.Sdk`, not the older Functions SDK. | [Isolated worker guide](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide), [supported languages](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages) |
| MAUI 10.0.120 (2026-10-21). MAUI 10 support ends 2027-05-11. MAUI 9 is out of support. | Holds. Policy page last updated 2026-10-22. Latest patch is still 10.0.120, released 2026-10-21. End of support 2027-05-11. MAUI 9 ended 2026-05-12. The next .NET Patch Tuesday is 2026-11-10, so 10.0.120 can stop being the latest servicing build within a week. Microsoft support requires the latest servicing update. | [MAUI support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/maui) |
| Azure Static Web Apps hosts React, and Microsoft’s deploy tutorial scaffolds it with Vite (`npm create vite@latest --template react`). | Holds for hosting. The React pivot of the deploy tutorial still runs `npm create vite@latest swa-react-demo -- --template react`. That template is the JavaScript `react` preset, not `react-ts`. The same page’s Node prerequisite is still “20.0 or later,” which is stale: Node 20 is off the current LTS table, and Vite 8 requires Node 20.19+ or 22.12+. | [Deploy a web framework](https://learn.microsoft.com/en-us/azure/static-web-apps/deploy-web-framework), [Vite getting started](https://vite.dev/guide/) |
| Cosmos free tier is 1000 RU/s and 25 GB, one account per subscription, chosen at creation, and is not available on serverless. | Holds on the free-tier article and the portal create flow. Serverless hides “Apply free tier discount.” The pricing FAQ still has a second answer that says the first 400 RU/s and 5 GB are free. That sentence conflicts with the free-tier article. The article is the one the log cited, and it still says 1000 RU/s and 25 GB for the life of the account. | [Free tier](https://learn.microsoft.com/en-us/azure/cosmos-db/free-tier), [create account](https://learn.microsoft.com/en-us/azure/cosmos-db/how-to-manage-database-account) |
| Cosmos vNext emulator is GA as of 2026-06-02. Image `mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator:vnext-latest`. Gateway mode, NoSQL, HTTPS required for the .NET SDK, auth not enforced, subset of features. | Holds. GA post is dated 2026-06-02 and pulls that tag. Learn still documents the tag, `--protocol https` for the .NET SDK, Data Explorer on port 1234, and no authentication enforcement. Batch API is listed as supported, so AD-2’s transactional batch is in the supported subset. Preview-era etag bugs inside transactional batch were reported fixed on `vnext-latest`. | [GA announcement](https://devblogs.microsoft.com/cosmosdb/announcing-general-availability-of-the-azure-cosmos-db-vnext-emulator/), [Linux emulator](https://learn.microsoft.com/en-us/azure/cosmos-db/emulator-linux) |

Related fit that the log implied and that still matches the docs:

- Flex Consumption scale-to-zero is real. Always-ready instances are optional and the documented default is 0. Creating the app without that setting bills only while functions execute. AD-13’s “no always-ready instances” matches that default. Source: [Flex Consumption plan](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan).
- .NET 11 is not a replacement pin. The download page lists 11.0.0-rc.3 (2026-10-25) as a go-live release candidate, not GA. .NET 10 remains the current LTS.
- MAUI 10’s bare `net10.0-android` target defaults to Android API 36. Google Play has required API 36 for new apps and updates since 2026-08-31; the extension window closed 2026-11-01. The memlog’s Play Store decision fits MAUI 10. The spine does not mention the target API, and the log never checked it. Sources: [.NET 10 Android 36.1.2 notes](https://github.com/dotnet/android/releases/tag/36.1.2), [Play target API](https://support.google.com/googleplay/android-developer/answer/11926878?hl=en).

## Live starter the spine leans on

Memlog, 2026-11-04: “React is the Vite TypeScript starter.” That starter is `npm create vite@latest -- --template react-ts`. Its `package.json` on `vitejs/vite` `main`, read 2026-11-04:

| Field | Live `template-react-ts` |
| --- | --- |
| `react`, `react-dom` | `^19.3.1` |
| `typescript` | `~7.0.2` |
| `vite` | `^8.3.5` |
| `@types/node` | `^24.19.2` |
| `@vitejs/plugin-react` | `^6.1.2` |
| lint | `oxlint` `^1.90.0` |
| test runner | none |

Source: [template-react-ts/package.json](https://raw.githubusercontent.com/vitejs/vite/main/packages/create-vite/template-react-ts/package.json).

npm `latest` the same day: `react` 19.3.1 (2026-10-14), `vite` 8.3.5. TypeScript 7.0 shipped 2026-07-08; 7.0.2 is the current stable (2026-08-20). TypeScript 7.1 is in beta (announced 2026-10-20), not the starter pin. Vite’s own docs: Node.js 20.19+ or 22.12+.

Vitest is not created by that starter. Current Vitest is 5.x. Vitest 5.1 was announced 2026-11-02. Vitest 5 requires Vite >= 6.4 and Node >= 22.12. Sources: [Vitest guide](https://vitest.dev/guide/), [Vitest 5.1](https://vitest.dev/blog/vitest-5-1).

xUnit is also unnamed by version. The current line is xUnit.net v3. Templates package `xunit.v3.templates` 4.0.2 was published 2026-10-09. The documented scaffold is `dotnet new install xunit.v3.templates` then `dotnet new xunit3`. The v3 getting-started page (examples from 2026-05, SDK 10.0.102) still shows a default `TargetFramework` of `net8.0` unless the framework choice is passed. v3 supports net10.0 as a template choice. The older in-box `dotnet new xunit` (v2) was not what that getting-started uses. Source: [xUnit v3 getting started](https://xunit.net/docs/getting-started/v3/getting-started), [xunit.v3.templates 4.0.2](https://www.nuget.org/packages/xunit.v3.templates/).

## Findings

### 1. Node.js 22 was not checked, and it is not the starter’s live default

- **Location:** Stack table, “Node.js | 22”. Structural seed comment “Vite, React, TypeScript.”
- **Trigger:** No `(version)` line mentions Node. The live `react-ts` starter depends on `@types/node` `^24.19.2`. Node 26 is Active LTS (since 2026-10-28, end of life 2029-04-30). Node 22 is Maintenance LTS and ends 2027-04-30, before .NET 10’s 2028-11-14. Vite 8 rejects Node 22 below 22.12. Vitest 5 rejects Node below 22.12. A pin of “22” includes 22.0–22.11, which do not run this starter.
- **Guard:** Re-check Node against the starter and the [Node release schedule](https://github.com/nodejs/Release). Either pin a floor the starter accepts (`22.12` or later on the 22 line) or move to the Active LTS the new work should track (26), and record the check.
- **Consequence:** An implementer on an early Node 22 install cannot run Vite 8 or Vitest 5. An implementer who trusts the stack table over `npm create vite` fights the starter’s Node 24 types.

### 2. TypeScript 7 is the starter default and is absent from the stack

- **Location:** Stack table (no TypeScript row). Seed: `web/` is “Vite, React, TypeScript.”
- **Trigger:** The log never records a TypeScript version. The live starter pins `typescript` `~7.0.2`. TypeScript 7.0 (2026-07-08) is the native Go compiler, a different toolchain from the TypeScript 5.x an agent would remember. Stable npm is 7.0.2; 7.1 is beta only.
- **Guard:** Add the starter’s TypeScript pin to the stack after a logged check, or say the web project is whatever `create-vite` `react-ts` emits and stop naming a partial set of its dependencies.
- **Consequence:** A builder who installs TypeScript 5.x, or who follows editor setup for the old compiler, does not get the project the starter creates. The spine and the starter then disagree on the language toolchain.

### 3. Vitest is required and is not in the starter, and it has no version

- **Location:** AD-10, AD-11, capability map. Stack table has no Vitest row.
- **Trigger:** The log says Vitest runs the shared JSON examples. It records no version and no check. `template-react-ts` has no test script and no Vitest dependency. Adding it today means Vitest 5, which needs Node >= 22.12 and Vite >= 6.4 (Vite 8 qualifies).
- **Guard:** Treat Vitest as an added dependency. Pin the major that was actually installed or resolved, and record the Node floor with it.
- **Consequence:** “Use the Vite TypeScript starter” does not produce the test runner AD-10 and AD-11 require. Guessing an older Vitest major fails peer requirements on Vite 8.

### 4. xUnit is unnamed across a breaking line change

- **Location:** AD-11. Stack table has no xUnit row.
- **Trigger:** No version check. Current xUnit for a new .NET 10 test project is v3 (`xunit3`, templates 4.0.2, 2026-10-09). v3 projects are executables on the Microsoft Testing Platform. The published v3 sample still defaults the template to `net8.0` unless a framework is selected. The in-box v2 template is a different product under the same name.
- **Guard:** Name v3, and record `dotnet new xunit3` with an explicit `net10.0` framework choice after checking the template’s default on the machine that will scaffold.
- **Consequence:** `dotnet new xunit` can emit the v2 stack, or `dotnet new xunit3` can emit `net8.0`, which is two weeks from end of support (2026-11-10). Either result breaks the “.NET 10” stack without the spine noticing.

### 5. React 19 and Vite 8 match the starter, but they were never logged

- **Location:** Stack table, “React | 19” and “Vite | 8”.
- **Trigger:** The only web check in the log is that Static Web Apps still has a Vite React tutorial. That tutorial uses `--template react` (JavaScript). It does not establish React 19 or Vite 8. Both majors are correct today: starter and npm latest are React `^19.3.1` and Vite `^8.3.5` (docs site is the Vite 8 guide; Node floor 20.19+ / 22.12+).
- **Guard:** Add a version line that cites the starter `package.json` and npm `latest`, including the patch or the range the starter uses, not only the major.
- **Consequence:** The majors happen to be right. The next reader cannot tell that from the memlog, and the Microsoft tutorial the log cites does not pin them. The same gap is how Node 22 and a missing TypeScript row got into the spine.

## Checked and still a fit, not logged

These named technologies exist and match the decisions. They had no version line. They do not contradict the spine.

| Decision | Check on 2026-11-04 |
| --- | --- |
| ASP.NET Core problem-details on the Function App (AD-8) | Available when the isolated worker uses ASP.NET Core integration (`ConfigureFunctionsWebApplication` and `Microsoft.Azure.Functions.Worker.Extensions.Http.AspNetCore`). Microsoft’s current .NET 10 migration sample references that package at 2.1.0. The plain isolated HTTP model does not give that pipeline. The spine is right only if the API uses the ASP.NET Core integration. |
| Cosmos transactional batch in one partition (AD-2), run locally on vNext (AD-13) | Learn’s feature table marks Batch API supported on the vNext emulator. Auth is still not enforced, which the log already says. |
| Google ID token, audience, `sub` (AD-3) | Not a versioned library in the stack. Not re-litigated here beyond noting the log never fetched a current Google sign-in doc. No spine pin depends on a Google SDK version. |
| Static Web Apps “current React host” | The product still deploys a Vite `dist` build. The older getting-started React tab still says build output `build`, which is the Create React App layout. The deploy-framework tutorial is the one that matches Vite. |

## Out of date in the sources, not in the spine

- Static Web Apps deploy tutorial: “Node.js 20.0 or later.” Vite 8’s own floor is 20.19+ or 22.12+, and Node 20 is no longer on the LTS schedule. Do not copy that prerequisite into the stack.
- Cosmos pricing FAQ: one answer still says 400 RU/s and 5 GB. The free-tier article still says 1000 RU/s and 25 GB. The spine follows the article.
- xUnit v3 getting-started examples are dated 2026-05 and show `net8.0` as the default framework. Confirm the default on `xunit.v3.templates` 4.0.2 before scaffolding.
