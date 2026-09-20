---
name: edna-reference-curation-assignment-workflow
description: 'Use when eDNA COI MOTU sequences require a traceable reference set assembled
  from public records, curated with lineage reconciliation, formatted for OBITools,
  and assigned to the most specific consistent taxon.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - co1-reference-sequence-assembly
  - taxonomic-lineage-retrieval-and-formatting
  - reference-database-validation-against-deposit
  - rdp-training-set-construction
  - coi-reference-database-curation
  - fasta-reference-database-curation
  - bold-c-oi-sequence-retrieval-replace
  - coi-reference-database-formatting
  - cross-release-reference-database-comparison
  - metabarcoding-taxonomic-assignment
  - obitools-ecotag-workflow
  - ecotaxonomic-assignment-table-processing
  - taxonomy-table-integration-with-read-counts
  member_tools:
  - BLAST
  - BioPerl
  - CO1Classifier repository releases
  - Ebot
  - Perl
  - RDP classifier
  - CO1Classifier (Porter)
  - QIIME 2
  - R
  - R (figure4_database_comparisons.R)
  - VSEARCH
  - bold (R package)
  - csvtk
  - seqkit
  - BOLD database
  - OBITools
  - ecoPCR
  - ecotag
  - sumaclust
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabarcoding/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# eDNA Reference Curation and Taxonomic Assignment

## Summary

Assemble, curate, and format a combined COI reference database, then use it to assign eDNA MOTU representatives while retaining read counts and ambiguity flags.


## When to use

Use when eDNA COI MOTU sequences require a traceable reference set assembled from public records, curated with lineage reconciliation, formatted for OBITools, and assigned to the most specific consistent taxon.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — assemble_reference

**Goal:** retrieve COI records, screen human contaminants, and preserve complete lineages

**EDAM operation:** http://edamontology.org/operation_0306

**Inputs:** tsv/target-taxa, database/ncbi-taxonomy, database/ncbi-nucleotide, database/human-co1-blast · **Outputs:** fasta/assembled-co1-reference, tsv/rdp-taxonomy, archive/rdp-classifier

**Candidate leaf skills:** `co1-reference-sequence-assembly` (primary), `taxonomic-lineage-retrieval-and-formatting`, `reference-database-validation-against-deposit`, `rdp-training-set-construction`

**Tools (primary):** BLAST, BioPerl, CO1Classifier repository releases, Ebot, Perl, RDP classifier

**Other candidate tools:** Perl + BioPerl (Ebot script), RDP classifier v2.12, Perl + BioPerl + Ebot

**Grounding:** 1 KB(s); DOIs: 10.1038/s41598-018-22505-4

### Stage 2 — curate_reference

**Goal:** filter taxonomy, dereplicate identical sequences, reconcile conflicts, and cluster centroids

**EDAM operation:** http://edamontology.org/operation_0291

**Inputs:** fasta/assembled-co1-reference, tsv/rdp-taxonomy, fasta/bold-coi-reference, tsv/taxonomy-exclusion-patterns · **Outputs:** fasta/curated-coi-reference, tsv/curated-taxonomy

**Candidate leaf skills:** `coi-reference-database-curation` (primary), `fasta-reference-database-curation`, `bold-c-oi-sequence-retrieval-replace`

**Tools (primary):** CO1Classifier (Porter), QIIME 2, R, R (figure4_database_comparisons.R), VSEARCH, bold (R package), csvtk, seqkit

**Other candidate tools:** custom R script, ecoPCR, OBITools

**Grounding:** 2 KB(s); DOIs: 10.1002/ece3.6594, 10.1038/s41598-017-17150-2

### Stage 3 — format_reference

**Goal:** build an OBITools-compatible combined COI reference index

**EDAM operation:** http://edamontology.org/operation_0335

**Inputs:** fasta/curated-coi-reference, tsv/curated-taxonomy, fasta/non-target-coi-reference · **Outputs:** database/obitools-coi-reference

**Candidate leaf skills:** `coi-reference-database-formatting` (primary), `cross-release-reference-database-comparison`

**Tools (primary):** BOLD database, OBITools, ecoPCR, ecotag

**Other candidate tools:** custom R script

**Grounding:** 1 KB(s); DOIs: 10.1038/s41598-017-17150-2

### Stage 4 — assign_motus

**Goal:** assign MOTUs to the most specific consistent taxon and retain ambiguous matches

**EDAM operation:** http://edamontology.org/operation_0346

**Inputs:** fasta/motu-representatives, database/obitools-coi-reference, feature-table/motu-counts · **Outputs:** tsv/motu-taxonomic-assignments, tsv/ambiguous-motus

**Candidate leaf skills:** `metabarcoding-taxonomic-assignment` (primary), `obitools-ecotag-workflow`, `ecotaxonomic-assignment-table-processing`, `taxonomy-table-integration-with-read-counts`

**Tools (primary):** OBITools, ecoPCR, ecotag, sumaclust

**Other candidate tools:** Reference-databases, obigrep

**Grounding:** 1 KB(s); DOIs: 10.1038/s41598-017-17150-2

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
