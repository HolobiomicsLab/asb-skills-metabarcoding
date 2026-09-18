---
name: sample-replicate-grouping-and-counting
description: Use when the input is an occurrence table of ASVs x samples x technical
  replicates with read counts and the goal is to remove non-reproducible occurrences
  by requiring presence in a minimum number of replicates.
when_to_use_negative:
- The input is raw FASTQ reads that have not yet been merged, demultiplexed, and run
  through VTAM's sortreads and filter commands to produce the Nijk occurrence table
- The analysis goal is to remove whole unreproducible replicates (FilterRenkonen,
  e.g. by Renkonen similarity) rather than individual non-reproducible occurrences
- The dataset contains no technical replicates per sample, since the minimum-replicate
  criterion cannot be evaluated
edam_operation: http://edamontology.org/operation_3695
tools:
- name: VTAM
  role: Source pipeline whose filter step implements FilterMinReplicateNumber to eliminate
    non-reproducible occurrences across replicates
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language for the reconstructed grouping/counting of occurrences
    by ASV-sample replicate presence; VTAM itself is implemented in Python
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:e7b9babc484808600007cbff354afcf8a8e7f5c196cc3c0a0c9c0ed6e7af3ea7
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Sample replicate grouping and counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the VTAM FilterMinReplicateNumber logic: group a replicate-structured occurrence table (read counts Nijk of ASV i in sample j and replicate k) by ASV-sample pair, count distinct replicates with non-zero reads, and discard occurrences that fail a minimum-replicate threshold (benchmark: >= 2 replicates) to ensure repeatability of metabarcoding occurrences.

## When to use

Use when the input is an occurrence table of ASVs x samples x technical replicates with read counts and the goal is to remove non-reproducible occurrences by requiring presence in a minimum number of replicates.

## When NOT to use

- The input is raw FASTQ reads that have not yet been merged, demultiplexed, and run through VTAM's sortreads and filter commands to produce the Nijk occurrence table
- The analysis goal is to remove whole unreproducible replicates (FilterRenkonen, e.g. by Renkonen similarity) rather than individual non-reproducible occurrences
- The dataset contains no technical replicates per sample, since the minimum-replicate criterion cannot be evaluated

## Inputs

- Replicate-structured occurrence table from VTAM merge/sortreads/filter stages (columns: ASV id, sample id, replicate id, read count Nijk)

## Outputs

- occurrences_minreplicate_filtered.tsv: occurrences retained that appear in at least the minimum number of replicates (>= 2), preserving ASV, sample, replicate and read-count columns

## How to apply

Load the replicate-structured occurrence table produced by VTAM's merge, sortreads, and filter stages, where each row records Nijk (read count of ASV i in sample j and replicate k). Group occurrences by ASV-sample pair and count the number of distinct replicates in which the ASV is present with a non-zero read count. Discard occurrences whose replicate count is below the minimum replicate threshold; the VTAM benchmark uses at least 2 replicates per sample. Write the surviving occurrences, preserving the original ASV, sample, replicate, and read-count columns, to a filtered TSV (occurrences_minreplicate_filtered.tsv). The rationale is that VTAM explicitly uses technical replicates to ensure repeatability, addressing non-reproducible occurrences (FilterMinReplicateNumber) alongside whole-replicate removal (FilterRenkonen).

## Related tools

- **VTAM** (Source pipeline whose filter step implements FilterMinReplicateNumber to eliminate non-reproducible occurrences across replicates) — https://github.com/aitgon/vtam
- **Python** (Implementation language for the reconstructed grouping/counting of occurrences by ASV-sample replicate presence; VTAM itself is implemented in Python)

## Examples

```
occ.groupby(['asv_id','sample_id'])['replicate'].nunique().ge(2)
```

## Evaluation signals

- Every retained ASV-sample occurrence appears in at least the minimum number of replicates (>= 2) in occurrences_minreplicate_filtered.tsv
- All removed occurrences had replicate counts strictly below the minimum-replicate threshold
- The output preserves the original ASV, sample, replicate, and read-count columns with no altered values
- No occurrence is retained for an ASV-sample pair whose total non-zero-replicate count is below the threshold
- Row count of the output equals the number of input occurrences with per-ASV-sample distinct-replicate counts >= the threshold

## Limitations

- The exact replicate-acceptance threshold is pipeline-dependent; the article's discussion only says the variant must be present 'in at least a certain number of replicates', with the benchmark value of 2 used here
- This filter removes occurrence-level non-reproducibility but does not address inter-sample contamination, tag-jump, or chimeras, which VTAM handles via other filters (FilterChimera, FilterCodonStop, FilterIndel, FilterLNF, FilterPCRerror)

## Evidence

- [readme] VTAM is a Python metabarcoding package: [evidence span withheld — anchor status: external_doc]
