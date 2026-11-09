---
name: low-frequency-noise-thresholding
description: Decision support for choosing and applying low-frequency-noise cutoffs
  (read count, per-sample-replicate abundance, per-variant abundance) to ASV occurrence
  tables during VTAM's filter step, especially when negative controls and replicates
  inform threshold optimization.
when_to_use_negative:
- The goal is filtering PCR errors via sequence similarity — VTAM's FilterPCRerror
  discards occurrences when an ASV closely matches another ASV with a significantly
  higher number of reads, which is a distinct filter from LFN thresholding.
- Input is raw FASTQ reads rather than an ASV occurrence table — reads must first
  go through VTAM's merge, sortreads, and filter steps before occurrence-level cutoffs
  apply.
- Cutoffs have not been optimized for the dataset at hand — the paper's values (60
  reads, 0.002, 0.006) were optimized for the bat dataset and VTAM suggested different
  read-count cut-offs for other datasets (10 for fish, 40 for shark).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: VTAM
  role: Implements the FilterLNF filter and the optimize step that determines the
    LFN cutoffs using negative control and mock samples
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Language in which VTAM is implemented; used for tabular occurrence filtering
    and cutoff validation
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:5de0b0b0112953f66434b235b20d7e138cad95c007efcc0a641060276b9edfd1
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# low-frequency-noise-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements VTAM's FilterLNF (Filter Low Frequency Noise) step, which removes low-frequency ASV occurrences that likely reflect tag-jump, inter-sample contamination, or sequencing noise by applying three per-occurrence read-count and relative-abundance cutoffs.

## When to use

Decision support for choosing and applying low-frequency-noise cutoffs (read count, per-sample-replicate abundance, per-variant abundance) to ASV occurrence tables during VTAM's filter step, especially when negative controls and replicates inform threshold optimization.

## When NOT to use

- The goal is filtering PCR errors via sequence similarity — VTAM's FilterPCRerror discards occurrences when an ASV closely matches another ASV with a significantly higher number of reads, which is a distinct filter from LFN thresholding.
- Input is raw FASTQ reads rather than an ASV occurrence table — reads must first go through VTAM's merge, sortreads, and filter steps before occurrence-level cutoffs apply.
- Cutoffs have not been optimized for the dataset at hand — the paper's values (60 reads, 0.002, 0.006) were optimized for the bat dataset and VTAM suggested different read-count cut-offs for other datasets (10 for fish, 40 for shark).

## Inputs

- ASV occurrence table (ASV i, sample j, replicate k) with read counts Nijk, per-sample-replicate totals Njk, and per-ASV totals Ni
- VTAM filter-step output (occurrences from FASTQ-derived FASTA files after merge and sortreads)
- Optimized parameter values for lfn_read_count_cutoff, lfn_sample_replicate_cutoff, lfn_variant_cutoff (e.g. from VTAM's optimize step)

## Outputs

- filtered_occurrences.tsv — tab-separated table of surviving occurrences with ASV ID, sample, replicate and read count columns
- Set of eliminated low-frequency-noise occurrences

## How to apply

Load the ASV occurrence table with read counts Nijk (ASV i, sample j, replicate k), per-sample-replicate totals Njk, and per-ASV totals Ni. For each occurrence compute three criteria: absolute reads Nijk, relative abundance within the sample-replicate Nijk/Njk, and relative abundance within the ASV Nijk/Ni; remove the occurrence if Nijk < lfn_read_count_cutoff OR Nijk/Njk < lfn_sample_replicate_cutoff OR Nijk/Ni < lfn_variant_cutoff. For the VTAM benchmark's bat dataset (357 fecal pellet samples, 24 mock samples, 19 negative controls, 3 PCR replicates each), the optimized values were lfn_read_count_cutoff=60, lfn_sample_replicate_cutoff=0.002, and lfn_variant_cutoff=0.006, determined by the optimize step of VTAM which processes negative control and mock samples to minimize both false negatives and false positives. FilterLNF runs within VTAM's filter step alongside FilterPCRerror, so it should be applied to the occurrence table produced after merge, sortreads, and the initial filter input (FASTQ-derived FASTA files). Write surviving occurrences to filtered_occurrences.tsv preserving ASV ID, sample, replicate, and read count columns.

## Related tools

- **VTAM** (Implements the FilterLNF filter and the optimize step that determines the LFN cutoffs using negative control and mock samples) — https://github.com/aitgon/vtam
- **Python** (Language in which VTAM is implemented; used for tabular occurrence filtering and cutoff validation) — https://github.com/aitgon/vtam

## Examples

```
df = pd.read_csv('occurrences.tsv', sep='\t'); df[(df.Nijk>=60)&(df.Nijk/df.Njk>=0.002)&(df.Nijk/df.Ni>=0.006)].to_csv('filtered_occurrences.tsv', sep='\t', index=False)
```

## Evaluation signals

- No occurrence in filtered_occurrences.tsv violates any cutoff: all retained occurrences satisfy Nijk >= 60, Nijk/Njk >= 0.002, and Nijk/Ni >= 0.006 for the bat-dataset parameters.
- filtered_occurrences.tsv is a well-formed tab-separated table preserving ASV ID, sample, replicate and read count columns.
- Negative controls and mock samples are clean (or near-clean) after filtering, consistent with VTAM's principle that 'the best parameter setting that produces clean control samples can also be applied to real samples'.
- Cutoffs match those optimized by VTAM for the dataset being processed (60 for bat, 40 for shark, 10 for fish), not blindly copied across datasets.
- Comparing against FilterPCRerror output confirms the two filters eliminate distinct occurrence classes (similar low-read ASVs vs. low-frequency noise).

## Limitations

- Cutoffs are dataset-specific: the optimized bat-dataset values (60, 0.002, 0.006) are not transferable — VTAM suggested read-count cut-offs of 10 for fish and 40 for shark datasets.
- Denosing/clustering algorithms alone cannot account for inter-sample contamination, tag-jump, and chimeras, which is why LFN thresholding must be combined with the other VTAM filters (FilterPCRerror, FilterChimera, FilterMinReplicateNumber, FilterRenkonen).
- Optimization of lfn_sample_replicate_cutoff and pcr_error_var_prop parameters is independent of the other parameters, so parameter tuning order matters.
- The published benchmark paper does not report software versions, so exact FilterLNF behavior may vary across VTAM releases.

## Evidence

- [readme] VTAM is a Python metabarcoding package: [evidence span withheld — anchor status: external_doc]
