---
name: amplicon-asv-taxonomy-diversity-workflow
description: 'Use when demultiplexed paired-end Illumina amplicon reads must be denoised
  into QIIME 2 ASV artifacts, assigned taxonomy, and summarized with alpha and beta
  diversity statistics in one coherent Tourmaline workflow.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 3
  member_skills:
  - amplicon-sequence-denoising-dada2
  - asv-denoising
  - asv-feature-table-generation
  - dada2-asv-denoising-and-pair-merging
  - taxonomic-classification-assignment
  - qiime2-artifact-generation
  - taxonomic-assignment-amplicon-sequences
  - qiime2-artifact-import--a9b98430d1674072
  - alpha-beta-diversity-analysis
  - alpha-diversity-group-significance-comparison
  - permanova-group-significance-testing
  - non-rarefied-diversity-calculation
  member_tools:
  - DADA2
  - Docker
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
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

# Amplicon ASV Taxonomy and Diversity

## Summary

Convert raw paired-end amplicon reads into QIIME 2 feature-table, representative- sequence, taxonomy, distance, ordination, and group-significance outputs.


## When to use

Use when demultiplexed paired-end Illumina amplicon reads must be denoised into QIIME 2 ASV artifacts, assigned taxonomy, and summarized with alpha and beta diversity statistics in one coherent Tourmaline workflow.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — denoise_asvs

**Goal:** quality-filter, denoise, merge, and remove chimeras into QIIME 2 ASV artifacts

**EDAM operation:** http://edamontology.org/operation_3219

**Inputs:** paired-fastq/demultiplexed, tsv/sample-manifest, tsv/sample-metadata, yaml/qaqc-config · **Outputs:** qiime2-artifact/feature-table, qiime2-artifact/representative-sequences

**Candidate leaf skills:** `amplicon-sequence-denoising-dada2` (primary), `asv-denoising`, `asv-feature-table-generation`, `dada2-asv-denoising-and-pair-merging`

**Tools (primary):** DADA2, Docker, Python, QIIME 2, QIIME 2 viewer, Snakemake, Tourmaline

**Other candidate tools:** Phyloseq, argparser, ggplot2, mothur, Deblur, FastQC, BLAST, conda, cutadapt, dadasnake, seqtk

**Grounding:** 4 KB(s); DOIs: 10.1038/s41598-023-30764-z, 10.1093/bib/bbz155, 10.1093/gigascience/giaa135, 10.1093/gigascience/giac066

### Stage 2 — assign_taxonomy

**Goal:** assign reference-backed taxonomy to the representative-sequence artifact

**EDAM operation:** http://edamontology.org/operation_3460

**Inputs:** qiime2-artifact/representative-sequences, qiime2-artifact/taxonomy-reference, yaml/taxonomy-config · **Outputs:** qiime2-artifact/taxonomy

**Candidate leaf skills:** `taxonomic-classification-assignment` (primary), `qiime2-artifact-generation`, `taxonomic-assignment-amplicon-sequences`, `qiime2-artifact-import--a9b98430d1674072`

**Tools (primary):** DADA2, Python, QIIME 2, QIIME 2 viewer, Snakemake, Tourmaline

**Other candidate tools:** EasyAmplicon

**Grounding:** 3 KB(s); DOIs: 10.1002/imo2.42, 10.1007/s13238-020-00724-8, 10.1093/gigascience/giac066

### Stage 3 — analyze_diversity

**Goal:** calculate core diversity metrics, ordinations, and metadata-group significance

**EDAM operation:** http://edamontology.org/operation_2945

**Inputs:** qiime2-artifact/feature-table, qiime2-artifact/representative-sequences, qiime2-artifact/taxonomy, tsv/sample-metadata, integer/sampling-depth · **Outputs:** tsv/alpha-diversity, qiime2-artifact/distance-matrices, tsv/pcoa-coordinates, tsv/permanova-statistics

**Candidate leaf skills:** `alpha-beta-diversity-analysis` (primary), `alpha-diversity-group-significance-comparison`, `permanova-group-significance-testing`, `non-rarefied-diversity-calculation`

**Tools (primary):** DADA2, Python, QIIME 2, QIIME 2 viewer, Snakemake, Tourmaline

**Other candidate tools:** Jupyter notebooks, QIIME2, anaconda, sra-tools

**Grounding:** 2 KB(s); DOIs: 10.1038/s41598-024-77864-y, 10.1093/gigascience/giac066

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
