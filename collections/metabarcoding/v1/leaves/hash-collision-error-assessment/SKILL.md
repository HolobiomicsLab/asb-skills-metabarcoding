---
name: hash-collision-error-assessment
description: Use when estimating or replicating fastp's paired-end duplication rate
  from FASTQ data, implementing key1/key2 array-based duplication counting, or assessing
  how hash collisions bias duplication metrics versus FASTQC.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp/this key scheme targets
  short reads; use fastplong instead
- Goal is FASTQC-style hash-table duplication profiling rather than fastp's array-based
  key1/key2 evaluation
- Single-end-only deduplication workflow where the paired-end key1/key2 scheme does
  not apply
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Reference implementation of duplication-rate evaluation; its JSON report supplies
    the ground-truth duplication rate for comparison
  repo: https://github.com/OpenGene/fastp
- name: C++
  role: Language for implementing the key1/key2 array duplication evaluator mirroring
    fastp's internals
- name: Python
  role: Scripting language for computing key1/key2 from FASTQ reads and comparing
    rates against the fastp JSON report
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:e5932ed5f8b604b5bccec2543dc25e035ef760a81b1f9f5f425dc57ecbcc2495
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hash-collision-error-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs fastp's array-based paired-end duplication evaluation, in which key1 (first 12 bp of read1, offset into a 2^24-entry array D) and key2 (first 32 bp of read2, packed into a 64-bit integer) replace FASTQC's hash-table approach, and quantifies the hash-collision error (~0.01% of reads) inherent to fixed-size counting arrays.

## When to use

Use when estimating or replicating fastp's paired-end duplication rate from FASTQ data, implementing key1/key2 array-based duplication counting, or assessing how hash collisions bias duplication metrics versus FASTQC.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp/this key scheme targets short reads; use fastplong instead
- Goal is FASTQC-style hash-table duplication profiling rather than fastp's array-based key1/key2 evaluation
- Single-end-only deduplication workflow where the paired-end key1/key2 scheme does not apply

## Inputs

- gzip-compressed paired-end FASTQ files (e.g. in.R1.fq.gz and in.R2.fq.gz, Illumina/MGI short reads)
- fastp JSON report containing the reference duplication-rate field

## Outputs

- Implemented PE duplication rate estimate in [0, 1] based on arrays D and C
- Comparison of implemented rate vs. fastp JSON-reported duplication rate, with hash-collision tolerance (~0.01% of reads) recorded

## How to apply

Load a paired-end FASTQ dataset (gzip-compressed in.R1.fq.gz and in.R2.fq.gz) and compute key1 by packing the first 12 bases of each read1 into an integer offset that flags presence in a 2^24-entry duplication array D. Compute key2 by packing the first 32 bases of each read2 into a 64-bit long integer and increment the corresponding entry in counting array C only for pairs whose key1 position is already flagged, treating a read as duplicated only when all of its base pairs are identical to another. Aggregate array C to derive the duplication rate, then run fastp on the same input (default mode, or --dedup with dup_calc_accuracy level 3) to obtain its reference duplication rate from the JSON report. Compare the implemented estimator's rate against fastp's reported value; disagreement within the expected hash-collision tolerance (~0.01% of reads) confirms correct reconstruction. Rationale: fastp uses fixed-size arrays for O(1) lookups and speed in C++, trading exactness for the small collision rate that this skill explicitly quantifies.

## Related tools

- **fastp** (Reference implementation of duplication-rate evaluation; its JSON report supplies the ground-truth duplication rate for comparison) — https://github.com/OpenGene/fastp
- **C++** (Language for implementing the key1/key2 array duplication evaluator mirroring fastp's internals)
- **Python** (Scripting language for computing key1/key2 from FASTQ reads and comparing rates against the fastp JSON report)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp.
```

## Evaluation signals

- fastp JSON report (fastp.) exists and contains a duplication-rate field whose value lies in [0, 1]
- Implemented estimator's duplication rate agrees with fastp's reference value within the expected hash-collision tolerance (~0.01% of reads)
- Array D contains exactly 2^24 entries indexed by the 12-bp key1 offsets derived from read1
- Counting array C is incremented only for read pairs whose key1 position is already flagged and whose 64-bit key2 (first 32 bp of read2) matches
- A read pair is counted as duplicated only when all of its base pairs are identical to another pair

## Limitations

- Fixed-size arrays introduce hash collisions, causing approximately 0.01% of reads to be miscounted relative to exact hashing
- The estimator is approximate: a read is treated as duplicated only when all of its base pairs are identical to another, so partial duplicates are not flagged
- fastp is designed for short reads (Illumina NovaSeq, MGI); for long reads the different tool fastplong should be used

## Evidence

- [other] fastp computes two keys from a paired-end read: the first 12 bp of read1 serves as key1 (offset into array D) and the first 32 bp of read2 serves as key2: [evidence span withheld — anchor status: not_found]
