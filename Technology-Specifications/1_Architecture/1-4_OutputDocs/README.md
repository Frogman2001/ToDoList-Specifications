# Architecture OutputDocs

Finished architecture deliverables. See [Architecture](../README.md) for this phase and the [pipeline README](../../README.md) for the shared rules.

## What to put here

The completed architecture document, and any other finished artifact the architecture workflow produced for later phases to consume. That includes [ARCHITECTURE-SPINE.md](ARCHITECTURE-SPINE.md), the finished AD-1 through AD-15 contract, and [epics.md](epics.md), the accepted epics handoff. The projects spine and the file map stay in Working until their assumptions are accepted. Development is the next reader. QA may also copy from here when it needs the architecture directly.

## What not to do

- Do not put drafts, memlogs, or other scratch here. Those stay in [1-3_Working](../1-3_Working/README.md).
- Do not edit these files from Development or QA. Those phases copy what they need into their own InputDocs, and those copies are then frozen.
- Do not revise a file here in place after a later phase has copied it. A new finished version is a new file, or a deliberate replacement by the architect before downstream copies exist.

## Where the workflow reads and writes

The workflow reads [1-1_InputDocs](../1-1_InputDocs/README.md) and [1-2_ProcessedDocs](../1-2_ProcessedDocs/README.md), runs in [1-3_Working](../1-3_Working/README.md), and writes the finished result here.
