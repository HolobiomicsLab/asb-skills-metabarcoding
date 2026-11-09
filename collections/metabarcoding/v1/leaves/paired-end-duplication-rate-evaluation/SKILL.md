---
name: paired-end-duplication-rate-evaluation
description: Use when you need to estimate or reproduce the duplication rate of paired-end
  FASTQ data (e.g. in.R1.fq.gz/in.R2.fq.gz) and want the memory-efficient key/array
  method fastp uses rather than a hash-table approach.
when_to_use_negative:
- Data is long reads from Nanopore/PacBio/Cyclone — the README directs such data to
  fastplong, not fastp
- Only single-end FASTQ data is available — the key1/key2 scheme requires paired read1/read2
  files
- You need exact duplicate detection of full sequences rather than the sampled 12
  bp/32 bp key approximation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: Reference implementation providing the duplication rate to validate the reconstructed
    key/array estimator; duplication evaluation runs in default mode or with --dedup
    at dup_calc_accuracy level 3
  repo: https://github.com/OpenGene/fastp
- name: C++
  role: Language in which fastp is developed and in which the 64-bit key2 packing
    and array updates are implemented
- name: Python
  role: Alternative scripting language for implementing the key1/key2 computation
    and counting-array aggregation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:ee13b5f6cf66d8621ada5bda611dd7fb33c79faadf125bfead6e03f7dbe80ac7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OpenGene/fastp
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - C++
  - Python
  - fastp
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# paired-end-duplication-rate-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs fastp's array-based paired-end duplication evaluation, which computes key1 from the first 12 bp of read1 and key2 from the first 32 bp of read2 (a 64-bit packed integer) and uses flagging/counting arrays instead of FASTQC-style hash tables to estimate the PE duplication rate.

## When to use

Use when you need to estimate or reproduce the duplication rate of paired-end FASTQ data (e.g. in.R1.fq.gz/in.R2.fq.gz) and want the memory-efficient key/array method fastp uses rather than a hash-table approach.

## When NOT to use

- Data is long reads from Nanopore/PacBio/Cyclone — the README directs such data to fastplong, not fastp
- Only single-end FASTQ data is available — the key1/key2 scheme requires paired read1/read2 files
- You need exact duplicate detection of full sequences rather than the sampled 12 bp/32 bp key approximation

## Inputs

- gzip-compressed paired-end FASTQ read1 file (e.g. in.R1.fq.gz)
- gzip-compressed paired-end FASTQ read2 file (e.g. in.R2.fq.gz)

## Outputs

- Estimated paired-end duplication rate in [0, 1]
- fastp JSON report containing a duplication-rate field (fastp.)

## How to apply

For each PE pair, pack the first 12 bases of read1 into an integer key1 used as an offset to flag presence in a 2^24-entry duplication array D. Pack the first 32 bases of read2 into a 64-bit long integer key2; for pairs whose key1 position is already flagged, increment the corresponding entry in counting array C. Aggregate C to estimate the duplication rate, counting a read as duplicated only when all of its base pairs are identical to another pair. Validate by running fastp with duplication evaluation enabled (default mode, or --dedup with dup_calc_accuracy level 3) and comparing your estimator's rate to the duplication-rate field in fastp's JSON report, allowing ~0.01% divergence from hash collisions.

## Related tools

- **fastp** (Reference implementation providing the duplication rate to validate the reconstructed key/array estimator; duplication evaluation runs in default mode or with --dedup at dup_calc_accuracy level 3) — https://github.com/OpenGene/fastp
- **C++** (Language in which fastp is developed and in which the 64-bit key2 packing and array updates are implemented)
- **Python** (Alternative scripting language for implementing the key1/key2 computation and counting-array aggregation)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp.
```

## Evaluation signals

- The output report fastp. exists and contains a duplication-rate field
- The duplication-rate value lies in the range 0-1
- The implemented estimator's rate agrees with fastp's reference value within ~0.01% (hash-collision tolerance)
- Array D (2^24 entries) is used only for key1 flagging, and counting array C only receives increments for pairs with an already-flagged key1 position
- Read pairs are counted as duplicates only when all base pairs are identical to another pair

## Limitations

- Hash collisions between different read pairs packing to the same key can perturb the estimated rate (~0.01% of reads tolerance)
- The 12 bp key1 / 32 bp key2 scheme is a sampled prefix approximation, not full-sequence comparison
- fastp is designed for short reads (Illumina NovaSeq, MGI); for long reads the README points to fastplong

## Evidence
