# QA

QA turns upstream specifications into test plans, cases, and acceptance checks. Read the [pipeline README](../README.md) for the rules shared by every phase.

## Inputs and output

Copy what this phase will use into [3-1_InputDocs](3-1_InputDocs/README.md). The default source is [Development OutputDocs](../2_Development/2-4_OutputDocs/README.md). Also copy any earlier frozen source the work needs: [Architecture OutputDocs](../1_Architecture/1-4_OutputDocs/README.md), or PO documents under [`_bmad-output/planning-artifacts/`](../../_bmad-output/planning-artifacts/). Copy only those files. Do not edit the sources or the copies.

Finished test plans, cases, and acceptance checks go in [3-4_OutputDocs](3-4_OutputDocs/README.md).

Run this phase’s BMAD workflow in [3-3_Working](3-3_Working/README.md). Record which inputs apply, and which parts apply, in [3-2_ProcessedDocs](3-2_ProcessedDocs/README.md) before or during that work.

## Stages

1. [3-1_InputDocs](3-1_InputDocs/README.md) — Frozen copies of development deliverables and any other upstream docs this phase will read.
2. [3-2_ProcessedDocs](3-2_ProcessedDocs/README.md) — Which of those files apply, and which parts apply.
3. [3-3_Working](3-3_Working/README.md) — The BMAD workflow for this phase: drafts, notes, and memlogs.
4. [3-4_OutputDocs](3-4_OutputDocs/README.md) — Finished test plans, cases, and acceptance checks only.

Do not work in Architecture or Development folders, and do not edit their OutputDocs.
