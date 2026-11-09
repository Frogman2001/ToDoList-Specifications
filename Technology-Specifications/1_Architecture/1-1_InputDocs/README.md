# Architecture InputDocs

Frozen copies of the product-owner documents the architect will read. See [Architecture](../README.md) for this phase and the [pipeline README](../../README.md) for the shared rules.

## What to put here

Copy files from [`_bmad-output/planning-artifacts/`](../../../_bmad-output/planning-artifacts/) that this phase will use: brief, PRD, addendums, UX `DESIGN.md`, `EXPERIENCE.md`, and any other PO planning doc the architect needs. Copy only those files. Keep the original filename so the source stays obvious.

Adding a new copy is allowed. A person may replace a snapshot on purpose when the phase should pick up an upstream change.

## What not to do

- Do not edit, reformat, annotate, or delete a file already here.
- Do not let a BMAD workflow write here.
- Do not patch a snapshot because the upstream document changed. Update the original PO document if a decision changed it, then leave this copy until someone refreshes it.
- Do not copy the whole planning tree. Copy only what this phase will use.

## Where the workflow reads and writes

Read these files. Write applicability notes to [1-2_ProcessedDocs](../1-2_ProcessedDocs/README.md). Run the workflow in [1-3_Working](../1-3_Working/README.md). Put the finished architecture document in [1-4_OutputDocs](../1-4_OutputDocs/README.md).
