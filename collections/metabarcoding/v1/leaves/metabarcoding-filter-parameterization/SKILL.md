---
name: metabarcoding-filter-parameterization
description: Use when ASV occurrence tables produced by VTAM's 'filter' command on
  demultiplexed FASTQ data (e.g. COI MFZR/ZFZR or 16S shark reads) must have filtering
  thresholds optimized per dataset from control samples.
when_to_use_negative:
- When no mock community samples or negative controls are present in the experimental
  design, since control samples are what drive the filtering parametrization
- When input is raw reads not yet processed by VTAM's 'merge'/'sortreads' commands
  — filtering is applied to ASV occurrence tables from the 'filter' step
- When applying FilterCodonStop/FilterIndel to non-coding markers such as 16S, where
  these pseudogene/spurious-sequence filters must be skipped (e.g. the shark dataset)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: VTAM
  role: Provides the 'filter', 'optimize', and 'make_known_occurrences' commands used
    to derive control-based filtering parameters (FilterLNF, FilterPCRerror, FilterMinReplicateNumber,
    FilterRenkonen, FilterChimera, FilterCodonStop, FilterIndel)
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM; the pipeline is installed via pip into a
    conda environment (python=3.9)
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:74720d322daefeeb352574142c3a2c27959adcc5aa0ce95c810ef960e5f21754
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
  - VTAM
  - metabaR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabarcoding-filter-parameterization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parameterize VTAM's metabarcoding filtering by explicitly using negative controls, mock community samples, and technical replicates to derive optimal cut-offs (FilterLNF-related lfn parameters, pcr_error_var_prop) that minimize false negatives and false positives, e.g. lfn_read_count_cutoff of 60 for bat, 10 for fish, and 40 for shark datasets.

## When to use

Use when ASV occurrence tables produced by VTAM's 'filter' command on demultiplexed FASTQ data (e.g. COI MFZR/ZFZR or 16S shark reads) must have filtering thresholds optimized per dataset from control samples.

## When NOT to use

- When no mock community samples or negative controls are present in the experimental design, since control samples are what drive the filtering parametrization
- When input is raw reads not yet processed by VTAM's 'merge'/'sortreads' commands — filtering is applied to ASV occurrence tables from the 'filter' step
- When applying FilterCodonStop/FilterIndel to non-coding markers such as 16S, where these pseudogene/spurious-sequence filters must be skipped (e.g. the shark dataset)

## Inputs

- Merged, demultiplexed, trimmed FASTA/FASTQ amplicon files (VTAM 'merge'/'sortreads' output) for bat, fish (MFZR and ZFZR markers separately), and shark datasets
- Sample/replicate metadata distinguishing mock community samples, negative controls, and true samples
- VTAM 'filter' output ASV occurrence tables run with default parameters
- known_occurrences.tsv (true positives in mocks, false positives in mocks and negative controls)

## Outputs

- Per-dataset parameter files containing lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, and lfn_read_count_cutoff values
- Optimized VTAM 'filter' runs yielding validated, control-cleaned ASV tables per dataset

## How to apply

Run VTAM's 'filter' command with default parameters on the merged, demultiplexed fasta/fastq files (produced by the 'merge' and 'sortreads' commands) for each dataset and marker. Build a known_occurrences.tsv file listing true positives in mock samples and false positives in mock and negative controls using VTAM's 'make_known_occurrences' command. Run the 'optimize' command, first solving lfn_sample_replicate_cutoff and pcr_error_var_prop since their optimization is independent of the other parameters, then determining lfn_variant_cutoff and lfn_read_count_cutoff by counting false positives and false negatives across parameter combinations and selecting the compromise. Apply the optimized values per dataset — bat: lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60; fish: 0.003, 0.1, 0.006, 10; shark: 0.001, 0.2, 0.016, 40 — and skip FilterCodonStop and FilterIndel for the shark dataset because 16S is non-coding.

## Related tools

- **VTAM** (Provides the 'filter', 'optimize', and 'make_known_occurrences' commands used to derive control-based filtering parameters (FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, FilterCodonStop, FilterIndel)) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM; the pipeline is installed via pip into a conda environment (python=3.9)) — https://github.com/aitgon/vtam

## Examples

```
vtam example; cd example; snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Each emitted parameter file exists and contains the four optimized cut-offs matching the reported per-dataset optima (lfn_read_count_cutoff 60 for bat, 10 for fish, 40 for shark)
- Optimized lfn_sample_replicate_cutoff and pcr_error_var_prop are determined first, since their optimization is independent of lfn_variant_cutoff and lfn_read_count_cutoff
- FilterCodonStop and FilterIndel are skipped for the shark dataset (16S non-coding marker)
- Filtering of the mock and negative controls yields no false positives while retaining expected true positives (minimizing both FN and FP)
- Taxonomic assignment uses the COInr COI database or RDP Classifier 16S trainsetNo18 via VTAM's 'taxassign' command so results are comparable across datasets

## Limitations

- Optimized cut-offs are dataset- and marker-specific; values derived for bat, fish (COI MFZR/ZFZR), or shark (16S) cannot be transferred wholesale to new datasets
- Commercial mock samples exist for bacteria and fungi, but metabarcoders studying metazoa or plants must construct adequate mock samples themselves
- The shark dataset requires skipping FilterCodonStop and FilterIndel because 16S is non-coding
- Only variants present in at least a certain number of replicates are kept, and the replicate-acceptance threshold must be set via the optimize step

## Evidence

- [readme] README quick working example invocation: [evidence span withheld — anchor status: external_doc]
