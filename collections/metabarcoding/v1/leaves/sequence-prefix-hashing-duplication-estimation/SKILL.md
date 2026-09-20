---
name: sequence-prefix-hashing-duplication-estimation
description: Use when a duplication rate must be estimated for paired-end FASTQ data
  or a custom deduplication estimator must be reproduced, specifically fastp's array-based
  key1/key2 scheme rather than FASTQC's hash-table method.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — the paper explicitly directs
  such data to fastplong instead of fastp
- Only single-end FASTQ is available — key1/key2 are defined from a paired-end read
  (12 bp of read1 + 32 bp of read2), so the scheme does not transfer directly
- Exact per-read duplicate IDs are needed rather than a rate estimate — the array-based
  scheme trades a small hash-collision error (~0.01% of reads) for speed
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: Reference implementation of the array-based duplication evaluation; its JSON
    report supplies the reference duplication rate for comparison
  repo: https://github.com/OpenGene/fastp
- name: C++
  role: Implementation language for packing 12 bp/32 bp sequence prefixes into integer
    keys and maintaining the 2^24-entry duplication array D and counting array C
- name: Python
  role: Alternative scripting language to compute key1/key2 offsets and aggregate
    duplication counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:68b1bd443f13775e9c9af292039d7564eabbb9f62f46598d59f5dc29dc98ce10
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
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# sequence-prefix-hashing-duplication-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements fastp's array-based paired-end duplication evaluation, in which the first 12 bp of read1 form key1 (an offset into a 2^24-entry flagging array D) and the first 32 bp of read2 are packed into a 64-bit key2 counted in array C — unlike FASTQC's hash-table approach.

## When to use

Use when a duplication rate must be estimated for paired-end FASTQ data or a custom deduplication estimator must be reproduced, specifically fastp's array-based key1/key2 scheme rather than FASTQC's hash-table method.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — the paper explicitly directs such data to fastplong instead of fastp
- Only single-end FASTQ is available — key1/key2 are defined from a paired-end read (12 bp of read1 + 32 bp of read2), so the scheme does not transfer directly
- Exact per-read duplicate IDs are needed rather than a rate estimate — the array-based scheme trades a small hash-collision error (~0.01% of reads) for speed

## Inputs

- gzip-compressed paired-end FASTQ read1 file (e.g. in.R1.fq.gz)
- gzip-compressed paired-end FASTQ read2 file (e.g. in.R2.fq.gz)

## Outputs

- Estimated duplication rate (float in 0-1) for the PE dataset
- fastp JSON report containing a duplication-rate field for cross-validation

## How to apply

Load a paired-end FASTQ dataset (e.g. gzip-compressed in.R1.fq.gz and in.R2.fq.gz). For each pair, pack the first 12 bases of read1 into an integer offset used to flag presence in a 2^24-entry duplication array D (key1). For pairs whose key1 slot is already flagged, pack the first 32 bases of read2 into a 64-bit integer (key2) and increment the corresponding entry in counting array C. Aggregate the counts into a duplication rate in [0,1], treating a read as duplicated only when all of its base pairs are identical to another read's. Validate by running fastp on the same input with duplication evaluation enabled (default mode, or --dedup with dup_calc_accuracy level 3) and comparing its JSON-reported duplication rate against the implemented estimator.

## Related tools

- **fastp** (Reference implementation of the array-based duplication evaluation; its JSON report supplies the reference duplication rate for comparison) — https://github.com/OpenGene/fastp
- **C++** (Implementation language for packing 12 bp/32 bp sequence prefixes into integer keys and maintaining the 2^24-entry duplication array D and counting array C)
- **Python** (Alternative scripting language to compute key1/key2 offsets and aggregate duplication counts)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp.
```

## Evaluation signals

- The fastp JSON output report exists and contains a duplication-rate field whose value lies in the range 0-1
- The implemented estimator's duplication rate agrees with fastp's reported value within the expected hash-collision tolerance (~0.01% of reads)
- A read pair is counted as duplicated only when all of its base pairs are identical to another pair
- key1 indexing respects the 2^24-entry array-D size (12 bp prefix), and key2 is encoded as a 64-bit long integer from the first 32 bp of read2

## Limitations

- Hash collisions introduce a small inaccuracy, on the order of ~0.01% of reads, versus exact duplicate detection
- The key1/key2 scheme is defined for paired-end short reads (12 bp of read1 plus 32 bp of read2) and does not generalize to long reads or single-end data
- fastp (and this estimator) is designed for short reads such as Illumina NovaSeq/MGI; long-read preprocessing requires fastplong

## Evidence

- [full_text] fastp computes two keys from a paired-end read: the first 12 bp of read1 serves as key1 (offset into array D) and the first 32 bp of read2 serves as key2: [evidence span withheld — anchor status: not_found]
