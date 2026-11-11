# Architecture

The architect turns product-owner planning into an architecture document. Read the [pipeline README](../README.md) for the rules shared by every phase.

## Inputs and output

Copy the PO documents this phase will use from [`_bmad-output/planning-artifacts/`](../../_bmad-output/planning-artifacts/) into [1-1_InputDocs](1-1_InputDocs/README.md). Typical sources are the brief, PRD, addendums, and UX `DESIGN.md` and `EXPERIENCE.md`. Copy only what applies. Do not edit the originals or the copies.

The finished architecture document, the AD-1 through AD-15 spine, and the epics handoff go in [1-4_OutputDocs](1-4_OutputDocs/README.md). Development and QA may copy them into their own InputDocs. They do not edit them here. The epics handoff is the architect's list of completable user outcomes. Development still writes the stories it will build.

Run the architecture BMAD workflow in [1-3_Working](1-3_Working/README.md). Record which inputs apply, and which parts apply, in [1-2_ProcessedDocs](1-2_ProcessedDocs/README.md) before or during that work.

## Stages

1. [1-1_InputDocs](1-1_InputDocs/README.md) — Frozen copies of the PO documents this phase will read.
2. [1-2_ProcessedDocs](1-2_ProcessedDocs/README.md) — Which of those files apply, and which parts apply.
3. [1-3_Working](1-3_Working/README.md) — The BMAD architecture workflow: drafts, notes, and memlogs.
4. [1-4_OutputDocs](1-4_OutputDocs/README.md) — The finished architecture document, the AD-1 through AD-15 spine, and, once accepted, the epics handoff.

Do not work in Development or QA folders. Do not write application code here.
