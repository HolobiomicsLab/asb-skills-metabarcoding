---
name: metabarcoding-occurrence-table-curation
description: Use when you have a replicate-structured ASV-by-sample occurrence table
  (read counts Nijk per ASV i, sample j, replicate k) plus merged fasta sequences
  from VTAM merge/sortreads, and you need to remove chimeras and non-reproducible
  occurrences.
when_to_use_negative:
- Input is already a curated ASV table that has passed VTAM's filter step (FilterLNF,
  FilterPCRerror, FilterChimera, FilterMinReplicateNumber) — re-running curation risks
  discarding valid occurrences
- The sequencing data have no technical replicates, so FilterMinReplicateNumber's
  replicate-count criterion cannot be evaluated
- A different denoising/clustering pipeline (DADA2, Swarm, Unoise, Deblur) already
  produced the ASV table and VTAM-specific chimera comparison logic is not applicable
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: VTAM
  role: Provides the 'filter' command implementing FilterChimera and FilterMinReplicateNumber
    over occurrence tables and merged fasta files
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM; used for the standalone filtering/validation
    scripts over the occurrence TSV tables
  repo: https://github.com/aitgon/vtam
- name: WopMars
  role: Workflow manager framework (database-driven, SQLAlchemy/SQlite models) within
    which VTAM's filtering steps are executed reproducibly
  repo: https://github.com/aitgon/wopmars
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:2f9adf8da1908660512ce1b5589b2e95bb75765f2d22e2b85506050a7308f57a
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# metabarcoding-occurrence-table-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Curate a metabarcoding ASV occurrence table by reconstructing VTAM's filtering sub-steps: removing chimeric ASVs (FilterChimera) and discarding non-reproducible occurrences across technical replicates (FilterMinReplicateNumber, benchmark threshold of at least 2 replicates). This addresses false positives such as chimeras and tag-jump/contamination artifacts that denoising/clustering algorithms alone cannot handle.

## When to use

Use when you have a replicate-structured ASV-by-sample occurrence table (read counts Nijk per ASV i, sample j, replicate k) plus merged fasta sequences from VTAM merge/sortreads, and you need to remove chimeras and non-reproducible occurrences.

## When NOT to use

- Input is already a curated ASV table that has passed VTAM's filter step (FilterLNF, FilterPCRerror, FilterChimera, FilterMinReplicateNumber) — re-running curation risks discarding valid occurrences
- The sequencing data have no technical replicates, so FilterMinReplicateNumber's replicate-count criterion cannot be evaluated
- A different denoising/clustering pipeline (DADA2, Swarm, Unoise, Deblur) already produced the ASV table and VTAM-specific chimera comparison logic is not applicable

## Inputs

- Replicate-structured ASV occurrence table (ASV i, sample j, replicate k, read count Nijk)
- Merged fasta ASV sequences from VTAM merge/sortreads steps (e.g. bat dataset, Galan et al. 2018, Dryad doi:10.5061/dryad.kv02g)
- VTAM filter configuration with FilterChimera and FilterMinReplicateNumber enabled (minimum replicate threshold, benchmark value >= 2)

## Outputs

- Chimera-filtered occurrence table with all occurrences of chimeric ASVs removed (TSV)
- Removed chimeric ASV ID list matching the chimera-flag list
- occurrences_minreplicate_filtered.tsv containing occurrences appearing in at least 2 replicates per sample

## How to apply

Load the occurrence table where each row records the read count of ASV i in sample j and replicate k (Nijk), produced by VTAM's merge and sortreads commands on the bat dataset (Galan et al. 2018, Dryad doi:10.5061/dryad.kv02g). Run the VTAM 'filter' command (Python3, WopMars framework) with FilterChimera enabled so each ASV is compared against more abundant ASVs to flag chimeric sequences; other filters (FilterCodonStop, FilterIndel) can be deactivated for this standalone operation, and extract the flagged chimera sequence IDs. Separately, group occurrences by ASV-sample pair, count the distinct replicates with non-zero read counts, and discard occurrences below the benchmark threshold of at least 2 replicates (FilterMinReplicateNumber). Rationale: denoising algorithms filter PCR/sequencing errors but cannot account for inter-sample contamination, tag-jump, and chimeras, while VTAM's explicit use of replicates ensures repeatability and control-sample-driven thresholds minimize both false negatives and false positives. Write the curated occurrence table as TSV, preserving ASV, sample, replicate and read-count columns.

## Related tools

- **VTAM** (Provides the 'filter' command implementing FilterChimera and FilterMinReplicateNumber over occurrence tables and merged fasta files) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM; used for the standalone filtering/validation scripts over the occurrence TSV tables) — https://github.com/aitgon/vtam
- **WopMars** (Workflow manager framework (database-driven, SQLAlchemy/SQlite models) within which VTAM's filtering steps are executed reproducibly) — https://github.com/aitgon/wopmars

## Examples

```
snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- No ASV flagged as a chimera by FilterChimera remains in the filtered occurrence table
- The set of removed ASV IDs exactly matches the chimera-flag list extracted from the filter output
- Every retained ASV-sample occurrence appears in at least 2 replicates in occurrences_minreplicate_filtered.tsv
- All removed occurrences had fewer than the minimum replicate threshold count
- Output TSV preserves the original ASV, sample, replicate and read-count (Nijk) column schema

## Limitations

- The benchmark minimum-replicate threshold (>= 2) is dataset-dependent; VTAM suggests different optimized cut-offs per dataset (read-count cut-offs 10 for fish, 40 for shark, 60 for bat)
- Denoising/clustering algorithms filter most PCR and sequencing errors but cannot account for inter-sample contamination, tag-jump, and chimeras, so filtering must be combined with control-sample-driven thresholds
- Optimal parameter settings (e.g. lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006 for the bat dataset) must be re-derived per experiment using negative controls and mock samples

## Evidence

- [readme] VTAM is a metabarcoding package with various commands to process high throughput sequencing (HTS) data of amplicons: [evidence span withheld — anchor status: external_doc]
