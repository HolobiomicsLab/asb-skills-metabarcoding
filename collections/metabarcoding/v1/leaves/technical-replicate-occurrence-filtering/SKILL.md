---
name: technical-replicate-occurrence-filtering
description: Use when the FASTQ-derived occurrence table is replicate-structured (each
  sample sequenced in >=2 PCR replicates) and non-reproducible ASV occurrences must
  be removed before downstream ecological analysis.
when_to_use_negative:
- Samples lack technical replicates (e.g. single-replicate designs), so replicate
  reproducibility cannot be assessed
- The input is already a filtered ASV-per-sample count table with no replicate dimension
  — applying a replicate threshold would be meaningless
- The goal is controlling contamination or index hopping, which in VTAM is handled
  by negative-control/mock-sample filters rather than the replicate filter
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: VTAM
  role: Provides the FilterMinReplicateNumber filter and the upstream merge/sortreads/filter
    stages that generate the replicate-structured occurrence table
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language for VTAM and for scripting the replicate-count filtering
    and validation of the occurrence table
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:460ccf2e2d5955762d399cd308f0d0c4bbd36e15daef0af8eb154a620c96df0a
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# technical-replicate-occurrence-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters metabarcoding occurrence tables (ASV i, sample j, replicate k with read count Nijk) by requiring each ASV-sample occurrence to appear in a minimum number of the sample's technical replicates, as implemented by VTAM's FilterMinReplicateNumber. Non-reproducible occurrences are discarded to ensure repeatability and reduce spurious detections.

## When to use

Use when the FASTQ-derived occurrence table is replicate-structured (each sample sequenced in >=2 PCR replicates) and non-reproducible ASV occurrences must be removed before downstream ecological analysis.

## When NOT to use

- Samples lack technical replicates (e.g. single-replicate designs), so replicate reproducibility cannot be assessed
- The input is already a filtered ASV-per-sample count table with no replicate dimension — applying a replicate threshold would be meaningless
- The goal is controlling contamination or index hopping, which in VTAM is handled by negative-control/mock-sample filters rather than the replicate filter

## Inputs

- Replicate-structured occurrence table with columns ASV, sample, replicate, and read count (Nijk), produced by VTAM merge/sortreads/filter stages
- VTAM filter-step output fasta files as upstream input
- Minimum replicate threshold parameter (benchmark value: >=2 replicates per sample)

## Outputs

- occurrences_minreplicate_filtered.tsv: occurrence table retaining only ASV-sample pairs observed in at least the minimum number of replicates, preserving ASV, sample, replicate, and read-count columns

## How to apply

Start from the replicate-structured occurrence table produced by VTAM's merge/sortreads/filter stages, where each row records read count Nijk for ASV i in sample j and replicate k. Group occurrences by ASV-sample pair and count the distinct replicates in which the ASV has a non-zero read count. Discard occurrences whose replicate count is below the minimum replicate threshold — the VTAM benchmark uses at least 2 replicates per sample (the FilterMinReplicateNumber step), which can be combined with FilterRenkonen, which removes whole poorly reproducible replicates. Rationale: requiring occurrence reproducibility across replicates is one of VTAM's explicit experimental-design mechanisms (alongside negative controls and mock samples) for minimizing false positives while retaining sensitivity. Write surviving occurrences to an output table (e.g. occurrences_minreplicate_filtered.tsv) preserving the original ASV, sample, replicate, and read-count columns.

## Related tools

- **VTAM** (Provides the FilterMinReplicateNumber filter and the upstream merge/sortreads/filter stages that generate the replicate-structured occurrence table) — https://github.com/aitgon/vtam
- **Python** (Implementation language for VTAM and for scripting the replicate-count filtering and validation of the occurrence table) — https://github.com/aitgon/vtam

## Examples

```
cd example; snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Every retained ASV-sample pair appears in at least the minimum number of replicates (>=2) in the output table
- Every removed occurrence had a replicate count strictly below the threshold
- Output table preserves the original ASV, sample, replicate, and read-count column schema
- No read counts were altered — only rows dropped (counts in retained rows match the input)
- Positive controls/mock samples remain clean after filtering, consistent with VTAM's parameter-optimization principle

## Limitations

- Only a 'certain number of replicates' threshold is documented in general terms; the exact acceptance rule must be fixed per study (benchmark used >=2 replicates)
- FilterMinReplicateNumber targets non-reproducible occurrences only; whole poorly reproducible replicates require the separate FilterRenkonen filter
- The replicate threshold trades sensitivity for precision — too strict a cutoff can remove true low-abundance ASVs that fail to amplify consistently

## Evidence

- [readme] VTAM produces ASV tables from HTS amplicon data: [evidence span withheld — anchor status: external_doc]
