# Development

The developer turns the architecture into specs, stories, and plans that implementation and QA can follow. Read the [pipeline README](../README.md) for the rules shared by every phase.

Application code is not written here. It stays in the ToDoList repository. This phase produces specification documents.

## Inputs and output

Copy what this phase will use into [2-1_InputDocs](2-1_InputDocs/README.md). The default source is [Architecture OutputDocs](../1_Architecture/1-4_OutputDocs/README.md). Also copy any earlier frozen source the work needs: PO documents under [`_bmad-output/planning-artifacts/`](../../_bmad-output/planning-artifacts/), or any other earlier OutputDocs. Copy only those files. Do not edit the sources or the copies.

Finished specs, stories, and plans go in [2-4_OutputDocs](2-4_OutputDocs/README.md). QA copies from there into its own InputDocs. QA does not edit these files.

Run this phase’s BMAD workflow in [2-3_Working](2-3_Working/README.md). Record which inputs apply, and which parts apply, in [2-2_ProcessedDocs](2-2_ProcessedDocs/README.md) before or during that work.

## Stages

1. [2-1_InputDocs](2-1_InputDocs/README.md) — Frozen copies of the architecture document and any other upstream docs this phase will read.
2. [2-2_ProcessedDocs](2-2_ProcessedDocs/README.md) — Which of those files apply, and which parts apply.
3. [2-3_Working](2-3_Working/README.md) — The BMAD workflow for this phase: drafts, notes, and memlogs.
4. [2-4_OutputDocs](2-4_OutputDocs/README.md) — Finished specs, stories, and plans only.

Do not work in Architecture or QA folders, and do not edit their OutputDocs.
