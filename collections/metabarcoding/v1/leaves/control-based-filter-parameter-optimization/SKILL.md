---
name: control-based-filter-parameter-optimization
description: Use when processing amplicon HTS metabarcoding data (FASTQ, ASV occurrence
  tables) that include negative controls and/or mock samples, and you need data-driven
  filter cutoffs instead of arbitrary read-count thresholds.
when_to_use_negative:
- The dataset has no negative controls or mock samples, since VTAM's parametrization
  depends explicitly on control samples to minimize FN and FP
- You only need generic denoising/clustering of sequencing errors (Swarm, Unoise,
  Deblur, DADA2) without control-driven filter threshold optimization
- Input is already a curated, filter-thresholded ASV table rather than raw occurrences
  with read counts per replicate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: VTAM
  role: Runs the filter and optimize commands that use negative control and mock samples
    to determine optimal filtering parameters and eliminate occurrences
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Language in which VTAM is implemented; used for downstream validation of filtered
    occurrence tables
- name: WopMars
  role: Workflow manager used to run the reproducible VTAM filtering pipeline
  repo: github.com/aitgon/wopmars
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:f03bfb3907266d6bdfdecaed7ad66c86cf3d9caee5faeb436b5072a45fe34581
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
  - Python
  - VTAM
  - WopMars
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# control-based-filter-parameter-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Explicitly process negative control and mock community samples (with technical replicates) to determine optimal metabarcoding filtering parameter settings — e.g. VTAM's lfn_read_count_cutoff, lfn_sample_replicate_cutoff, lfn_variant_cutoff and pcr_error_var_prop — that minimize both false negatives (FN) and false positives (FP).

## When to use

Use when processing amplicon HTS metabarcoding data (FASTQ, ASV occurrence tables) that include negative controls and/or mock samples, and you need data-driven filter cutoffs instead of arbitrary read-count thresholds.

## When NOT to use

- The dataset has no negative controls or mock samples, since VTAM's parametrization depends explicitly on control samples to minimize FN and FP
- You only need generic denoising/clustering of sequencing errors (Swarm, Unoise, Deblur, DADA2) without control-driven filter threshold optimization
- Input is already a curated, filter-thresholded ASV table rather than raw occurrences with read counts per replicate

## Inputs

- FASTQ amplicon reads (merged, demultiplexed, primer-trimmed by VTAM merge/sortreads)
- ASV occurrence table (ASV i, sample j, replicate k) with read counts Nijk, per-replicate totals Njk and per-ASV totals Ni
- Experimental design / sample metadata distinguishing negative controls, mock community samples and PCR replicates

## Outputs

- Optimized filter parameter values (e.g. lfn_read_count_cutoff=60, lfn_sample_replicate_cutoff=0.002, lfn_variant_cutoff=0.006, pcr_error_var_prop=0.2)
- Filtered occurrence table (e.g. filtered_occurrences.tsv with ASV ID, sample, replicate and read-count columns)

## How to apply

In VTAM, run the filter step on the FASTQ-derived occurrence table and then the optimize command, which searches parameter combinations so that negative controls and mock samples come out clean while sensitivity is retained. Optimize lfn_sample_replicate_cutoff and pcr_error_var_prop independently of the other parameters, then fix lfn_read_count_cutoff and lfn_variant_cutoff (the benchmark found optimized bat-dataset values of lfn_read_count_cutoff=60, lfn_sample_replicate_cutoff=0.002, lfn_variant_cutoff=0.006 and pcr_error_var_prop=0.2, with suggested read-count cut-offs of 10 for fish, 40 for shark and 60 for bat datasets). Apply the retained parameters through the occurrence filters (e.g. FilterLNF removes an occurrence when Nijk < 60, Nijk/Njk < 0.002 or Nijk/Ni < 0.006) and prepare known occurrences with make_known_occurrences for downstream replicate filtering. The rationale is that a parameter setting producing clean negative controls and mock samples can also be safely applied to real samples, yielding higher precision than pipelines that do not use controls for parametrization.

## Related tools

- **VTAM** (Runs the filter and optimize commands that use negative control and mock samples to determine optimal filtering parameters and eliminate occurrences) — https://github.com/aitgon/vtam
- **Python** (Language in which VTAM is implemented; used for downstream validation of filtered occurrence tables)
- **WopMars** (Workflow manager used to run the reproducible VTAM filtering pipeline) — github.com/aitgon/wopmars

## Examples

```
vtam example; cd example; snakemake --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- All occurrences surviving FilterLNF satisfy all three cutoffs: Nijk >= 60, Nijk/Njk >= 0.002 and Nijk/Ni >= 0.006
- Negative controls and mock samples come out clean (no false-positive ASVs) at the chosen parameter setting
- filtered_occurrences.tsv is a well-formed tab-separated table retaining ASV ID, sample, replicate and read-count columns
- Optimization of lfn_sample_replicate_cutoff and pcr_error_var_prop is performed independently of the other parameters, per the VTAM protocol
- Resulting precision is comparable to or better than fixed read-count cut-off baselines (e.g. OBIbaR with minimum read count cut-off 60 on the fish and bat datasets)

## Limitations

- Optimized cut-off values are dataset-dependent (10 for fish, 40 for shark, 60 for bat) and must be re-optimized per dataset
- Benchmark comparisons did not state software versions or dataset accession identifiers, limiting exact reproducibility
- Denoising/clustering algorithms alone cannot account for inter-sample contamination, tag-jump and chimeras, so control-based filtering must complement them

## Evidence

- [readme] VTAM tool identity and scope: [evidence span withheld — anchor status: external_doc]
