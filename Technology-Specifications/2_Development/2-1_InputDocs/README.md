# Development InputDocs

Frozen copies of the upstream documents the developer will read. See [Development](../README.md) for this phase and the [pipeline README](../../README.md) for the shared rules.

## What to put here

Copy the finished architecture document from [Architecture OutputDocs](../../1_Architecture/1-4_OutputDocs/README.md). That is the default input.

Also copy any earlier frozen source this phase will use. Allowed sources:

- PO planning docs in [`_bmad-output/planning-artifacts/`](../../../_bmad-output/planning-artifacts/) (brief, PRD, addendums, UX `DESIGN.md`, `EXPERIENCE.md`)
- OutputDocs of any earlier phase

Copy only the files this phase will use. Keep the original filename so the source stays obvious. Do not copy from an earlier phase’s Working folder.

Adding a new copy is allowed. A person may replace a snapshot on purpose when the phase should pick up an upstream change.

## What not to do

- Do not edit, reformat, annotate, or delete a file already here.
- Do not let a BMAD workflow write here.
- Do not edit Architecture OutputDocs or the PO originals. Copy them.
- Do not patch a snapshot because the upstream document changed. Leave this copy until someone refreshes it.

## Where the workflow reads and writes

Read these files. Write applicability notes to [2-2_ProcessedDocs](../2-2_ProcessedDocs/README.md). Run the workflow in [2-3_Working](../2-3_Working/README.md). Put finished specs, stories, and plans in [2-4_OutputDocs](../2-4_OutputDocs/README.md).
