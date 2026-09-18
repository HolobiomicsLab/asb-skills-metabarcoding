---
name: multi-marker-taxonomic-assignment
description: Use when you have an ASV/variant table from COI or 16S metabarcoding
  (e.g., VTAM filter output or DADA2/OBITools3 tables from the same markers) and need
  homogeneous taxonomic assignment across pipelines or markers.
when_to_use_negative:
- When the research question targets assignment-method performance itself — the benchmark
  explicitly fixed taxassign across all pipelines to ensure homogeneity, not to compare
  assigners
- When the marker is neither COI nor 16S and no VTAM-formatted reference database
  is available
- When input is raw FASTQ rather than a filtered ASV/variant table — taxassign is
  a downstream step after merge/sortreads/filter
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3050
tools:
- name: VTAM
  role: Provides the 'taxassign' command that performs taxonomic assignment for all
    three pipelines in the benchmark
  repo: https://github.com/aitgon/vtam
- name: Snakemake
  role: Orchestrates the vtam_benchmark workflow (02snkfl_analysis_vtam.yml) that
    runs filter followed by taxassign
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:33d3b3c451de89813d04ac9a3f5f8e1e1dd3021edd177023ef7371af4aa36293
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aitgon/vtam_benchmark
  repo_url: https://github.com/aitgon/vtam
  tools:
  - DADA2
  - LULU
  - OBITools3
  - Python
  - R
  - Snakemake
  - VTAM
  - cutadapt
  - metabaR
  - vsearch
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# multi-marker-taxonomic-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign taxonomy to filtered metabarcoding ASVs across different markers (COI and 16S) using VTAM's 'taxassign' command with marker-specific reference databases (COInr COI, RDP Classifier 16S trainsetNo18). Applied uniformly to all compared pipelines (VTAM, DALU, OBIbaR) so that downstream precision/sensitivity benchmarking is not confounded by differing assignment methods.

## When to use

Use when you have an ASV/variant table from COI or 16S metabarcoding (e.g., VTAM filter output or DADA2/OBITools3 tables from the same markers) and need homogeneous taxonomic assignment across pipelines or markers.

## When NOT to use

- When the research question targets assignment-method performance itself — the benchmark explicitly fixed taxassign across all pipelines to ensure homogeneity, not to compare assigners
- When the marker is neither COI nor 16S and no VTAM-formatted reference database is available
- When input is raw FASTQ rather than a filtered ASV/variant table — taxassign is a downstream step after merge/sortreads/filter

## Inputs

- Filtered ASV/variant tables (VTAM filter output; DADA2 denoised tables from DALU; OBITools3 obi uniq/grep/clean tables from OBIbaR)
- COInr COI reference database formatted for VTAM
- RDP Classifier 16S trainsetNo18 reference dataset formatted for VTAM (https://osf.io/ns2w8/)

## Outputs

- ASV table with taxonomic assignments (asvtable_taxa) for each pipeline and dataset
- Taxonomy-annotated occurrence tables used as input to add_filters.pl count_false_occ for TP/FP/FN classification

## How to apply

Prepare marker-appropriate reference databases in VTAM format: the COInr COI database for COI amplicons and the RDP Classifier 16S trainsetNo18 formatted for VTAM for 16S amplicons. Run VTAM's 'taxassign' command on the output of the filter step (using make_known_occurrences-derived known occurrences where applicable) to assign each ASV to taxa. Crucially, run taxassign for all pipelines being compared — DALU (DADA2 1.12.1 + LULU 0.1.0) and OBIbaR (OBITools3 3.0.1b19 + metabaR 1.0.0) included — so differences in abundance tables reflect filtering, not assignment. In the benchmark, taxassign was run after the filter step via the Snakemake workflow 02snkfl_analysis_vtam.yml, and assignment output feeds the false/true-positive classification (count_false_occ of add_filters.pl) and sensitivity/precision computation. The rationale is that VTAM is both a filtering and taxonomic-assignment pipeline ('Validation and Taxonomic Assignment of Metabarcoding data'), so using a single assignment step isolates the effect of control-based filtering parametrization.

## Related tools

- **VTAM** (Provides the 'taxassign' command that performs taxonomic assignment for all three pipelines in the benchmark) — https://github.com/aitgon/vtam
- **Snakemake** (Orchestrates the vtam_benchmark workflow (02snkfl_analysis_vtam.yml) that runs filter followed by taxassign)

## Examples

```
snakemake --printshellcmds --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Taxonomic assignments are produced for all three pipelines (VTAM, DALU, OBIbaR) using the same taxassign command, ensuring cross-pipeline comparability
- COI ASVs are assigned against COInr and 16S ASVs against RDPClassifier_16S_trainsetNo18 — verify the correct reference database was used per marker
- Downstream control_counts.tsv (from add_filters.pl count_false_occ) can be computed, confirming the assigned table is compatible with TP/FP classification
- Sensitivity TP/(TP+FN) and precision TP/(TP+FP) are computable from the assigned occurrences, matching the benchmark's metrics

## Limitations

- Assignment quality depends on reference database completeness; the article notes metabarcoders of metazoa or plants must construct adequate mock/reference communities since commercial mocks exist mainly for bacteria and fungi
- Software versions of some benchmark components are not stated in the article, affecting exact reproducibility

## Evidence

- [readme] snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa: [evidence span withheld — anchor status: external_doc]
