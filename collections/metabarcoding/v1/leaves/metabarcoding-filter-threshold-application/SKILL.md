---
name: metabarcoding-filter-threshold-application
description: Use when an ASV occurrence table (read counts per sample-replicate) contains
  low-frequency ASVs that closely match a much more abundant ASV and PCR/sequencing-error
  removal is needed before downstream filtering (replicates, chimeras, pseudogenes).
when_to_use_negative:
- Input is raw FASTQ reads rather than an ASV occurrence table — VTAM's merge, sortreads
  and filter steps must run first
- The goal is to address contamination, tag-jumps or chimeras rather than PCR/sequencing
  errors — the article states denoising/clustering algorithms filter PCR and sequencing
  errors but cannot account for inter-sample contamination, tag-jump, and chimeras
  (use FilterChimera or control-sample-based filters instead)
- A different VTAM filter is needed (e.g. FilterLNF, FilterMinReplicateNumber, FilterRenkonen,
  FilterCodonStop, FilterIndel), each targeting a distinct pitfall
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: VTAM
  role: Metabarcoding pipeline whose FilterPCRerror filter discards occurrences if
    an ASV closely matches another ASV with a significantly higher number of reads
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM; used to load the occurrence table with pandas
    and apply the pcr_error_var_prop proportion rule
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:218a95f3bee40414beb3b0528fb82a36ee339ec5ed5355c026aa2018d8dcbc11
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabarcoding-filter-threshold-application

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the VTAM FilterPCRerror step, which discards ASV occurrences whose read counts are a low proportion of those of a closely matching, far more abundant ASV, treating them as likely PCR/sequencing errors. It is the mechanism by which the VTAM pipeline addresses PCR errors among eDNA metabarcoding pitfalls.

## When to use

Use when an ASV occurrence table (read counts per sample-replicate) contains low-frequency ASVs that closely match a much more abundant ASV and PCR/sequencing-error removal is needed before downstream filtering (replicates, chimeras, pseudogenes).

## When NOT to use

- Input is raw FASTQ reads rather than an ASV occurrence table — VTAM's merge, sortreads and filter steps must run first
- The goal is to address contamination, tag-jumps or chimeras rather than PCR/sequencing errors — the article states denoising/clustering algorithms filter PCR and sequencing errors but cannot account for inter-sample contamination, tag-jump, and chimeras (use FilterChimera or control-sample-based filters instead)
- A different VTAM filter is needed (e.g. FilterLNF, FilterMinReplicateNumber, FilterRenkonen, FilterCodonStop, FilterIndel), each targeting a distinct pitfall

## Inputs

- ASV occurrence table with read counts per sample-replicate (e.g. vtam_bat_final_taxa.tsv from the VTAM benchmark)
- ASV sequences (for pairwise similarity comparison to identify closely matching more abundant ASVs)
- pcr_error_var_prop threshold parameter (0.2 for the optimized bat dataset)

## Outputs

- Filtered ASV occurrence table in TSV format (filtered_asv_occurrences.tsv) with PCR-error occurrences removed

## How to apply

Load the ASV occurrence table (read counts of ASVs per sample-replicate) — in the VTAM benchmark this is the taxa-annotated output vtam_bat_final_taxa.tsv — with pandas in Python. Compute pairwise similarity among ASV sequences to find, for each ASV, a closely matching ASV with a significantly higher total read count. Apply the FilterPCRerror rule: for each occurrence (ASV i, sample j, replicate k), discard it when its read count is a low proportion of the more abundant matching ASV's reads, i.e. below the pcr_error_var_prop cutoff (optimized to 0.2 for the bat dataset; the same VTAM optimize step sets lfn_sample_replicate_cutoff 0.002, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60). Write surviving occurrences to a TSV (filtered_asv_occurrences.tsv) and validate that removals correspond exclusively to ASVs closely matching a more abundant ASV below the proportion cutoff.

## Related tools

- **VTAM** (Metabarcoding pipeline whose FilterPCRerror filter discards occurrences if an ASV closely matches another ASV with a significantly higher number of reads) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM; used to load the occurrence table with pandas and apply the pcr_error_var_prop proportion rule) — https://github.com/aitgon/vtam

## Examples

```
import pandas as pd; df = pd.read_csv('vtam_bat_final_taxa.tsv', sep='\t')
```

## Evaluation signals

- filtered_asv_occurrences.tsv exists and parses as a valid TSV with the expected occurrence-table columns
- Every removed occurrence corresponds to an ASV that closely matches another ASV with a significantly higher number of reads
- Removed occurrences have read-count proportions below the pcr_error_var_prop cutoff of 0.2 relative to the more abundant matching ASV
- Surviving occurrences are unchanged relative to the input vtam_bat_final_taxa.tsv except for the discarded PCR-error rows
- The threshold value matches the VTAM optimize step output for the dataset (pcr_error_var_prop: 0.2 for the bat dataset)

## Limitations

- The optimized cutoffs (pcr_error_var_prop 0.2, lfn_read_count_cutoff 60, lfn_sample_replicate_cutoff 0.002, lfn_variant_cutoff 0.006) were tuned per dataset (bat dataset in this case) and should be re-optimized via the VTAM optimize command for other datasets
- FilterPCRerror addresses only PCR/sequencing errors among ASVs; it cannot account for inter-sample contamination, tag-jump, and chimeras
- Benchmark comparison values (sensitivity/precision) and software versions of the compared pipelines are not reported in the reviewed sections

## Evidence
