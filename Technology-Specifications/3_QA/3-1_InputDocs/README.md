# QA InputDocs

Frozen copies of the upstream documents QA will read. See [QA](../README.md) for this phase and the [pipeline README](../../README.md) for the shared rules.

## What to put here

Copy the finished specs, stories, and plans from [Development OutputDocs](../../2_Development/2-4_OutputDocs/README.md). That is the default input.

Also copy any earlier frozen source this phase will use. Allowed sources:

- [Architecture OutputDocs](../../1_Architecture/1-4_OutputDocs/README.md)
- PO planning docs in [`_bmad-output/planning-artifacts/`](../../../_bmad-output/planning-artifacts/) (brief, PRD, addendums, UX `DESIGN.md`, `EXPERIENCE.md`)
- OutputDocs of any earlier phase

Copy only the files this phase will use. Keep the original filename so the source stays obvious. Do not copy from an earlier phase’s Working folder.

Adding a new copy is allowed. A person may replace a snapshot on purpose when the phase should pick up an upstream change.

## What not to do

- Do not edit, reformat, annotate, or delete a file already here.
- Do not let a BMAD workflow write here.
- Do not edit Development OutputDocs, Architecture OutputDocs, or the PO originals. Copy them.
- Do not patch a snapshot because the upstream document changed. Leave this copy until someone refreshes it.

## Where the workflow reads and writes

Read these files. Write applicability notes to [3-2_ProcessedDocs](../3-2_ProcessedDocs/README.md). Run the workflow in [3-3_Working](../3-3_Working/README.md). Put finished test plans, cases, and acceptance checks in [3-4_OutputDocs](../3-4_OutputDocs/README.md).
