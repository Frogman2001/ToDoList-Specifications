# Technology specifications

This tree is the handoff pipeline after the product owner finishes planning. PO documents stay in [`_bmad-output/planning-artifacts/`](../_bmad-output/planning-artifacts/) (brief, PRD, addendums, UX `DESIGN.md` and `EXPERIENCE.md`). Do not move or rewrite them from here.

Three phases run in order. Each phase has the same four stages.

| Phase | Role | Default inputs | Deliverable |
| --- | --- | --- | --- |
| [1_Architecture](1_Architecture/README.md) | Architect | PO planning docs | Architecture document, AD-1 through AD-15 spine, and epics handoff |
| [2_Development](2_Development/README.md) | Developer | Architecture OutputDocs, plus any earlier source this phase needs | Specs, stories, and plans |
| [3_QA](3_QA/README.md) | QA | Development OutputDocs, plus Architecture OutputDocs or PO docs when needed | Test plans, cases, and acceptance checks |

Application code stays in the ToDoList repository. This tree holds specification documents only.

## Four stages

Each phase folder contains these stages. Read the stage README before writing there.

1. **InputDocs** — Frozen copies of upstream documents this phase will read. Copy only what this phase will use. Agents must not edit, reformat, or let a BMAD workflow write here. If an upstream document changes, do not patch the snapshot. A person may add or replace a copy on purpose.
2. **ProcessedDocs** — This phase’s analysis of which input files apply and which parts apply. Editable. This is not the BMAD run and not the deliverable.
3. **Working** — Where this phase’s BMAD workflow runs. Drafts, notes, and memlogs stay here.
4. **OutputDocs** — Finished deliverables only. The next phase copies from here into its own InputDocs. Later phases do not edit this folder.

A phase may copy from any earlier frozen source: the PO artifacts under `_bmad-output/planning-artifacts/`, and the OutputDocs of any earlier phase. Copy into this phase’s InputDocs. Do not edit the source.

## Rules for agents

- Spell the method **BMAD**. The folder name is `ProcessedDocs`.
- Do not work in another phase’s folders. Copy what you need, then work in the current phase.
- Read InputDocs and ProcessedDocs. Run the workflow in Working. Write the finished result to OutputDocs.
- Do not treat Working files as the handoff.

If work in this tree changes a decision already written in a PO document, update that original document and its memlog in the same session so the documents do not disagree. Leave the InputDocs snapshot unchanged until someone deliberately refreshes the copy.
