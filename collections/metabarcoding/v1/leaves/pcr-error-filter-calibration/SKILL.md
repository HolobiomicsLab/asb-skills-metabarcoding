---
name: pcr-error-filter-calibration
description: Use when processing merged, demultiplexed ASV occurrence tables from
  VTAM filter runs and needing control-derived, per-dataset PCR-error/low-frequency-noise
  cutoffs (e.g. pcr_error_var_prop 0.2 vs 0.1, lfn_read_count_cutoff 60/10/40) to
  minimize false positives and false negatives.
when_to_use_negative:
- Input is raw, unmerged FASTQ that has not passed VTAM merge/sortreads and a default
  filter run — optimize needs filtered ASV occurrence tables and known_occurrences.tsv
- 'The target marker is non-coding 16S (e.g. the shark dataset): FilterCodonStop and
  FilterIndel must be skipped, so a COI-tuned parameter set cannot be copied over
  without recalibration'
- No control samples (mocks or negatives) are available — the calibration is driven
  explicitly by control-sample occurrences and cannot be justified otherwise
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: VTAM
  role: Provides the filter, make_known_occurrences, and optimize commands used to
    derive and apply the per-dataset PCR-error/LFN cutoffs
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM; the package is installed and run via pip/conda
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:2729419daa72bf96f75194736ad0c91615262092c9ac53a60a46a2c8fd0933e2
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
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pcr-error-filter-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates metabarcoding PCR-error and low-frequency-noise filter parameters with VTAM's optimize command, using known occurrences in mock samples and negative/positive controls to set pcr_error_var_prop and lfn (lfn_sample_replicate_cutoff, lfn_variant_cutoff, lfn_read_count_cutoff) per dataset.

## When to use

Use when processing merged, demultiplexed ASV occurrence tables from VTAM filter runs and needing control-derived, per-dataset PCR-error/low-frequency-noise cutoffs (e.g. pcr_error_var_prop 0.2 vs 0.1, lfn_read_count_cutoff 60/10/40) to minimize false positives and false negatives.

## When NOT to use

- Input is raw, unmerged FASTQ that has not passed VTAM merge/sortreads and a default filter run — optimize needs filtered ASV occurrence tables and known_occurrences.tsv
- The target marker is non-coding 16S (e.g. the shark dataset): FilterCodonStop and FilterIndel must be skipped, so a COI-tuned parameter set cannot be copied over without recalibration
- No control samples (mocks or negatives) are available — the calibration is driven explicitly by control-sample occurrences and cannot be justified otherwise

## Inputs

- Merged, demultiplexed, primer/tag-trimmed FASTQ files (VTAM merge and sortreads output)
- ASV occurrence tables from VTAM filter run with default parameters
- known_occurrences.tsv built with VTAM make_known_occurrences (true positives in mocks, false positives in mock and negative controls)
- Sample/marker metadata (fastqinfo, readinfo, fastainfo tables)

## Outputs

- One optimized-parameter file per dataset with lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, lfn_read_count_cutoff (bat: 0.002/0.2/0.006/60; fish: 0.003/0.1/0.006/10; shark: 0.001/0.2/0.016/40)
- Optimize output tables of false positives and false negatives across parameter combinations

## How to apply

Run VTAM filter with default parameters on the merged, demultiplexed FASTQ data (merge and sortreads outputs) for each dataset, then build known_occurrences.tsv with make_known_occurrences, listing true positives in mock samples and false positives in mock and negative controls. Run vtam optimize first for lfn_sample_replicate_cutoff and pcr_error_var_prop because their optimization is independent of the other parameters, using the frequency of unexpected variants in mock samples and the proportion of closely matching expected/unexpected ASV read counts (FilterPCRerror discards occurrences whose ASV closely matches a much-higher-read ASV). Next determine lfn_variant_cutoff and lfn_read_count_cutoff by counting false positives and false negatives across parameter combinations and selecting the compromise; for 16S (non-coding) datasets such as shark, skip FilterCodonStop and FilterIndel. Write the optimized values per dataset into one parameter file each: lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60 for bat; 0.003, 0.1, 0.006, 10 for fish; 0.001, 0.2, 0.016, 40 for shark.

## Related tools

- **VTAM** (Provides the filter, make_known_occurrences, and optimize commands used to derive and apply the per-dataset PCR-error/LFN cutoffs) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM; the package is installed and run via pip/conda) — https://github.com/aitgon/vtam

## Examples

```
vtam example && cd example && snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Each emitted parameter file exists and contains exactly the four optimized cutoff values matching the reported per-dataset optima (e.g. lfn_read_count_cutoff 60 for bat, 10 for fish, 40 for shark)
- pcr_error_var_prop and lfn_sample_replicate_cutoff were optimized independently before lfn_variant_cutoff and lfn_read_count_cutoff, as VTAM's optimize workflow requires
- known_occurrences.tsv contains true positives from mock samples and false positives from mock and negative controls derived from the default-parameter filter output
- For the shark (16S, non-coding) dataset, FilterCodonStop and FilterIndel were skipped during optimization
- Filtered output should show higher precision (fewer false positives) without losing sensitivity, consistent with VTAM's reported behavior versus other pipelines

## Limitations

- Optimized cutoffs are dataset- and marker-specific (COI vs 16S) and cannot be transferred blindly; the shark 16S dataset requires skipping FilterCodonStop and FilterIndel
- The procedure depends on well-constructed mock community samples; for poorly represented taxonomic groups (e.g. metazoa or plants) commercial mocks are often unavailable
- The chosen lfn_read_count_cutoff is a compromise between false positives and false negatives, not a global optimum

## Evidence

- [methods] Per-dataset read count cut-offs derived by VTAM: [evidence span withheld — anchor status: not_found]
- [readme] VTAM processes HTS amplicon data into taxonomically assigned ASV tables: [evidence span withheld — anchor status: external_doc]
