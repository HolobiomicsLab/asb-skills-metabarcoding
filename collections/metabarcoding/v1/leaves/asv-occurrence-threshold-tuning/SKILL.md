---
name: asv-occurrence-threshold-tuning
description: Use when processing merged ASV occurrence tables from metabarcoding datasets
  (e.g. bat, fish, shark COI/16S) and you need per-dataset filtering cut-offs that
  minimize false positives and false negatives using control samples.
when_to_use_negative:
- Datasets lacking mock community samples or negative controls, since VTAM's optimize
  step derives thresholds explicitly from known occurrences in control samples
- Raw FASTQ input that has not yet passed through VTAM's merge, sortreads, and filter
  steps, which are prerequisites for building the ASV occurrence tables
- Datasets where a single global threshold is required across samples, since this
  skill produces per-dataset (and per-marker) optima rather than one universal cut-off
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3295
tools:
- name: VTAM
  role: Provides the filter, make_known_occurrences, and optimize commands used to
    derive per-dataset filtering thresholds from control samples
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM; optimize and parameter-file handling are
    executed through the Python-based VTAM package
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:79214340f287d9848ed8e13ddb0caeefd25b9a9e8c4603b7ef3f50c3d6145a7a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# asv-occurrence-threshold-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tunes VTAM's ASV occurrence filtering thresholds (lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, lfn_read_count_cutoff) per dataset by explicitly using mock, negative control, and replicate samples, yielding dataset-specific optima such as read count cut-offs of 60 (bat), 10 (fish), and 40 (shark).

## When to use

Use when processing merged ASV occurrence tables from metabarcoding datasets (e.g. bat, fish, shark COI/16S) and you need per-dataset filtering cut-offs that minimize false positives and false negatives using control samples.

## When NOT to use

- Datasets lacking mock community samples or negative controls, since VTAM's optimize step derives thresholds explicitly from known occurrences in control samples
- Raw FASTQ input that has not yet passed through VTAM's merge, sortreads, and filter steps, which are prerequisites for building the ASV occurrence tables
- Datasets where a single global threshold is required across samples, since this skill produces per-dataset (and per-marker) optima rather than one universal cut-off

## Inputs

- Merged, demultiplexed ASV occurrence tables from the VTAM filter command (run with default parameters)
- known_occurrences.tsv from VTAM make_known_occurrences (true positives in mock samples; false positives in mock and negative controls)
- Parameter combination grid over lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, lfn_read_count_cutoff
- Marker metadata distinguishing coding (COI) from non-coding (16S shark) markers

## Outputs

- One optimized parameter file per dataset listing lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, and lfn_read_count_cutoff (e.g. 0.002, 0.2, 0.006, 60 for the bat dataset)
- Dataset-specific read count cut-off values (60 for bat, 10 for fish, 40 for shark)

## How to apply

Run VTAM's filter command with default parameters to produce the demultiplexed, merged ASV occurrence tables, then build a known_occurrences.tsv file listing true positives in mock samples and false positives in mock and negative controls using VTAM's make_known_occurrences command. First optimize lfn_sample_replicate_cutoff and pcr_error_var_prop, since their optimization is independent of the other parameters, using the frequency of unexpected variants in mock samples and the proportion of closely matching expected/unexpected ASV read counts. Then determine lfn_variant_cutoff and lfn_read_count_cutoff by counting false positives and false negatives across a series of parameter combinations and selecting the compromise (skipping FilterCodonStop and FilterIndel for the non-coding 16S shark dataset). Write the four optimized values to one parameter file per dataset: 0.002/0.2/0.006/60 for bat, 0.003/0.1/0.006/10 for fish, and 0.001/0.2/0.016/40 for shark.

## Related tools

- **VTAM** (Provides the filter, make_known_occurrences, and optimize commands used to derive per-dataset filtering thresholds from control samples) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM; optimize and parameter-file handling are executed through the Python-based VTAM package) — https://github.com/aitgon/vtam

## Examples

```
snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Each emitted parameter file exists and contains the four optimized cutoff values matching the reported per-dataset optima (lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60 for bat; 0.003, 0.1, 0.006, 10 for fish; 0.001, 0.2, 0.016, 40 for shark)
- lfn_sample_replicate_cutoff and pcr_error_var_prop were determined before lfn_variant_cutoff and lfn_read_count_cutoff, consistent with their independence from the other parameters
- The chosen lfn_read_count_cutoff reflects a compromise between false positives and false negatives counted across the parameter-combination series
- For the shark 16S dataset, FilterCodonStop and FilterIndel were skipped during the optimization because 16S is non-coding
- Resulting read count cut-offs match VTAM's suggested values (10 for fish, 40 for shark, 60 for bat)

## Limitations

- Thresholds are dataset-specific: the same cut-off values do not transfer between the bat (60), fish (10), and shark (40) datasets
- For non-coding markers such as the shark 16S dataset, FilterCodonStop and FilterIndel must be skipped, altering the filter chain
- Commercial mock samples exist only for some taxonomic groups (bacteria and fungi); metabarcoders of metazoa or plants must construct adequate mock communities for the optimization to be valid

## Evidence
