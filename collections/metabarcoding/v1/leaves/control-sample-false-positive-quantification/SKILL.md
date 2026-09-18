---
name: control-sample-false-positive-quantification
description: Use when demultiplexed, merged ASV occurrence tables are available together
  with mock samples and negative controls, and you need to determine read-count or
  variant cut-offs that minimize false positives while retaining true positives.
when_to_use_negative:
- Input is a single-sample dataset without mock community samples or negative controls,
  since the optimization explicitly requires control samples to identify false positives
- The data have already been filtered with optimized parameters and only downstream
  taxonomic assignment (taxassign) is needed
- Non-metabarcoding amplicon data where replicate structure and control samples are
  absent, so lfn/replicate-based cut-offs cannot be estimated
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: VTAM
  role: Runs the filter, make_known_occurrences, and optimize commands used to quantify
    false positives in controls and derive per-dataset cut-offs
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM; the optimize/filter pipeline is executed
    via the Python-based VTAM package
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:3950c38ccd43adb528c9c22a3f3890a6b5762e44412cb1a8984aaffce061ca01
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
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# control-sample-false-positive-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies false positives in metabarcoding data using mock community and negative control samples to derive optimal filtering parameters (e.g. lfn_read_count_cutoff) per dataset with VTAM's optimize command.

## When to use

Use when demultiplexed, merged ASV occurrence tables are available together with mock samples and negative controls, and you need to determine read-count or variant cut-offs that minimize false positives while retaining true positives.

## When NOT to use

- Input is a single-sample dataset without mock community samples or negative controls, since the optimization explicitly requires control samples to identify false positives
- The data have already been filtered with optimized parameters and only downstream taxonomic assignment (taxassign) is needed
- Non-metabarcoding amplicon data where replicate structure and control samples are absent, so lfn/replicate-based cut-offs cannot be estimated

## Inputs

- Merged, demultiplexed, primer/tag-trimmed FASTQ files (output of VTAM merge and sortreads commands)
- ASV occurrence tables produced by the VTAM filter command with default parameters
- known_occurrences.tsv listing true positives in mock samples and false positives in mock and negative controls (from VTAM make_known_occurrences)
- Coded taxon expectations for mock community samples (bat, fish MFZR/ZFZR, shark datasets)

## Outputs

- One optimized parameter file per dataset containing lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, and lfn_read_count_cutoff (e.g. 0.002/0.2/0.006/60 for bat)
- Optimized lfn_read_count_cutoff values: 60 (bat), 10 (fish), 40 (shark)
- False-positive/false-negative counts across the scanned parameter combinations

## How to apply

First run VTAM's 'filter' command with default parameters on the merged, demultiplexed FASTQ data (produced by 'merge' and 'sortreads') to obtain ASV occurrence tables for the bat, fish (MFZR and ZFZR markers), and shark datasets. Prepare a known_occurrences.tsv file with the VTAM 'make_known_occurrences' command, listing true positives in mock samples and false positives in mock and negative controls. Run the 'optimize' command, first determining lfn_sample_replicate_cutoff and pcr_error_var_prop (since their optimization is independent of the other parameters), then scanning lfn_variant_cutoff and lfn_read_count_cutoff combinations to count false positives and false negatives and select the compromise. The per-dataset optima were: bat lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60; fish 0.003/0.1/0.006/10; shark 0.001/0.2/0.016/40. For the shark 16S dataset, skip FilterCodonStop and FilterIndel because 16S is non-coding. Write one parameter file per dataset with the four optimized cutoffs and apply them in the final filter run.

## Related tools

- **VTAM** (Runs the filter, make_known_occurrences, and optimize commands used to quantify false positives in controls and derive per-dataset cut-offs) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM; the optimize/filter pipeline is executed via the Python-based VTAM package) — https://github.com/aitgon/vtam

## Examples

```
vtam optimize --fastainput asvtable_filtered.fasta --asvtable asvtable_filtered.tsv --knownoccurrences known_occurrences.tsv --outfile optimize.csv
```

## Evaluation signals

- Each emitted parameter file exists and contains exactly the four optimized values matching the reported per-dataset optima (e.g. lfn_read_count_cutoff 60 for bat, 10 for fish, 40 for shark)
- known_occurrences.tsv distinguishes true positives in mock samples from false positives in mock and negative controls before optimization
- lfn_sample_replicate_cutoff and pcr_error_var_prop are optimized first, consistent with their independence from lfn_variant_cutoff and lfn_read_count_cutoff
- The chosen cut-off achieves a compromise of minimal false positives and false negatives across the scanned combinations
- For the shark 16S dataset, FilterCodonStop and FilterIndel are skipped, confirming the non-coding marker handling

## Limitations

- Optimized cut-offs are dataset-specific and must be re-derived per dataset; the same cut-off is not transferable (10 for fish vs 60 for bat)
- 16S datasets require skipping FilterCodonStop and FilterIndel, so the full filter stack does not apply to non-coding markers
- Adequate mock community construction is essential; commercial mocks exist mainly for bacteria and fungi, so metabarcoders of metazoa or plants must build their own

## Evidence
