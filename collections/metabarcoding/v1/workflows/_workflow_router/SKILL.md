---
name: metabarcoding-workflow-router
description: Use when a user has a whole metabarcoding analysis goal, such as processing amplicon reads through taxonomy and diversity or curating references for eDNA assignment, rather than a single step; select and run the appropriate end-to-end composite workflow with paper-grounded stages.
license: CC-BY-4.0
metadata:
  kind: workflow-router
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabarcoding/v1
  indexes: [workflows_index.json]
  bound_by: perspicacite-semantic
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# ASB Metabarcoding — composite workflow router

This is the **goal-level** entry point for the ASB Metabarcoding collection. The leaf router (`metabarcoding-router`) finds one atomic skill; this router selects an **end-to-end composite workflow** whose ordered stages delegate to vetted leaf skills with grounding and a declarative `workflow.yaml`.

## 1. Select — match the goal to a workflow

Prefer the collection's retrieval command:

```bash
python bin/search_skills.py --query "<the user's goal>" --collection . --target workflows --k 3
```

For manual lookup, load `workflows_index.json`, which has one row per composite workflow, and match the goal against each row's description and stages.

Available workflows (published as outlines):

| workflow | what it does |
|---|---|
| `amplicon-asv-taxonomy-diversity` | Convert raw paired-end amplicon reads into QIIME 2 feature-table, representative- sequence, taxonomy, distance, ordination, and group-significance outputs. |
| `edna-reference-curation-assignment` | Assemble, curate, and format a combined COI reference database, then use it to assign eDNA MOTU representatives while retaining read counts and ambiguity flags. |

If no workflow fits the goal, fall back to the leaf router (`metabarcoding-router`) and assemble steps from atomic skills.

## 2. Run — execute the workflow's stages

Read the chosen `workflows/<slug>/SKILL.md` and follow its **Stages** in order. Each stage carries a goal, candidate leaf skills with the primary first, tools to install or invoke, and typed inputs and outputs. The machine-readable `workflows/<slug>/workflow.yaml` is the DAG (`after`, `inputs_from`). Automatic grading of that DAG (`asb solve-workflow`) is **not part of this release**: no released ASB version loads these files, so run the stages yourself. Honor the I/O contract: each stage consumes the prior stage's declared outputs. Optional stages are marked.

For a stage's leaf skills, read each `leaves/<leaf-slug>/SKILL.md` for the procedure, or use the leaf router to pick among the candidates for the exact data.

## 3. Ground — verify each stage against its source papers

Before trusting a parameter or default, ground the stage's leaves against the papers from which they were distilled. Each stage's `grounding.kb_slugs` and `dois` in `workflow.yaml` name the source papers and the `asb-paper-<doi>` knowledge bases.

> These workflows are published as **outlines**: the stage structure is validated, the execution is not. Bindings were chosen by semantic retrieval plus deterministic selection. `derived_from_workflows` is a provenance record; no ablation experiment consuming it is released.
