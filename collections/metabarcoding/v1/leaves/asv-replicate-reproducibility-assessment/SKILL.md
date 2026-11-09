---
name: asv-replicate-reproducibility-assessment
description: Use when a metabarcoding occurrence table contains technical replicates
  per sample and non-reproducible ASV occurrences (e.g. tag-jumps, contaminants) must
  be removed by requiring occurrence repeatability across replicates.
when_to_use_negative:
- The sequencing design has only one replicate per sample, so occurrence reproducibility
  across replicates cannot be evaluated
- The goal is to remove whole poorly reproducible replicates (VTAM FilterRenkonen)
  rather than individual non-reproducible ASV-sample occurrences
- The occurrence table has already been replicate-filtered and you would risk double-filtering
  the same occurrences
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: VTAM
  role: Source of the FilterMinReplicateNumber filter; its merge, sortreads and filter
    stages produce the replicate-structured occurrence table that this assessment
    filters
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language for the replicate-counting and occurrence-discarding
    procedure over the occurrence table
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:931652e2a54c0beefa06d3045618741d18c83af1e60507614f962318575c11aa
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

# asv-replicate-reproducibility-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the VTAM FilterMinReplicateNumber logic: for each ASV-sample pair in a replicate-structured occurrence table (read counts Nijk per ASV i, sample j, replicate k), counts the distinct replicates with non-zero reads and discards occurrences not reproduced across at least a minimum number of technical replicates (benchmark threshold of 2).

## When to use

Use when a metabarcoding occurrence table contains technical replicates per sample and non-reproducible ASV occurrences (e.g. tag-jumps, contaminants) must be removed by requiring occurrence repeatability across replicates.

## When NOT to use

- The sequencing design has only one replicate per sample, so occurrence reproducibility across replicates cannot be evaluated
- The goal is to remove whole poorly reproducible replicates (VTAM FilterRenkonen) rather than individual non-reproducible ASV-sample occurrences
- The occurrence table has already been replicate-filtered and you would risk double-filtering the same occurrences

## Inputs

- Replicate-structured occurrence table (ASV i, sample j, replicate k, read count Nijk) from VTAM merge/sortreads/filter stages
- Minimum replicate threshold parameter (e.g. FilterMinReplicateNumber >= 2)

## Outputs

- occurrences_minreplicate_filtered.tsv with ASV, sample, replicate and read-count columns for occurrences present in at least the minimum number of replicates
- Set of discarded ASV-sample occurrences that appeared in fewer than the threshold number of replicates

## How to apply

Load the replicate-structured occurrence table where each row records the read count Nijk of an ASV i in sample j and replicate k, as produced by VTAM's merge, sortreads and filter stages. Group occurrences by ASV-sample pair and count the number of distinct replicates in which the ASV is present with a non-zero read count. Discard occurrences whose replicate count falls below the minimum replicate threshold (benchmark value: at least 2 replicates per sample, equivalent to VTAM's FilterMinReplicateNumber). Write the surviving occurrences, preserving the original ASV, sample, replicate and read-count columns, to occurrences_minreplicate_filtered.tsv. The rationale is that VTAM explicitly uses replicates to ensure repeatability, addressing non-reproducible occurrences that inflate false positives.

## Related tools

- **VTAM** (Source of the FilterMinReplicateNumber filter; its merge, sortreads and filter stages produce the replicate-structured occurrence table that this assessment filters) — https://github.com/aitgon/vtam
- **Python** (Implementation language for the replicate-counting and occurrence-discarding procedure over the occurrence table) — https://github.com/aitgon/vtam

## Examples

```
snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Every retained ASV-sample occurrence in occurrences_minreplicate_filtered.tsv appears in at least the minimum number of replicates (>= 2 distinct replicates with non-zero read counts)
- All removed ASV-sample occurrences had a replicate count strictly below the minimum replicate threshold
- The output table preserves the original ASV, sample, replicate and read-count columns with no modification of Nijk values
- The set of surviving occurrences is exactly the input table subset where replicate-count >= threshold, verifiable by re-grouping the input on (ASV, sample)

## Limitations

- Requires a replicate-structured experimental design; occurrences unique to a single replicate are lost, which can introduce false negatives for genuinely rare variants
- The exact 'certain number of replicates' threshold is dataset- and design-dependent; the benchmark used at least 2 replicates per sample, but the article notes numeric values for the replicate-acceptance threshold were not fully specified in all sections

## Evidence
