---
name: fastq-paired-end-data-handling
description: Use when processing paired-end Illumina FASTQ data (e.g. in.R1.fq.gz/in.R2.fq.gz)
  and you need duplication-rate evaluation or to replicate fastp's key1/key2 array-based
  duplication estimation.
when_to_use_negative:
- Input is single-end FASTQ — the key1/key2 scheme requires both mates of each pair
- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead, since
  fastp is designed for short reads (Illumina NovaSeq, MGI)
- Reads carry UMI sequences — deduplication should be UMI-aware rather than based
  on identical base pairs alone
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: Reference implementation of the array-based paired-end duplication evaluation;
    provides the duplication-rate value in its JSON report for comparison
  repo: https://github.com/OpenGene/fastp
- name: C++
  role: Implementation language used to re-implement the key1/key2 duplication arrays
    with fast integer packing
- name: Python
  role: Alternative language for scripting the duplication-key estimator over the
    FASTQ input
- name: fastplong
  role: fastp's counterpart for long reads; not applicable to this PE short-read skill
  repo: https://github.com/OpenGene/fastplong
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:f75864d9e86a1f5a105890613b8cd7585471014be09be2eddd03ed02710af32c
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
  - fastplong
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# fastq-paired-end-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs fastp's array-based paired-end duplication evaluation, where key1 is derived from the first 12 bp of read1 as an offset into a 2^24-entry duplication array D and key2 from the first 32 bp of read2 as a 64-bit integer incrementing a counting array C. This differs from FASTQC's hash-table approach and yields an estimate of the PE duplication rate.

## When to use

Use when processing paired-end Illumina FASTQ data (e.g. in.R1.fq.gz/in.R2.fq.gz) and you need duplication-rate evaluation or to replicate fastp's key1/key2 array-based duplication estimation.

## When NOT to use

- Input is single-end FASTQ — the key1/key2 scheme requires both mates of each pair
- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead, since fastp is designed for short reads (Illumina NovaSeq, MGI)
- Reads carry UMI sequences — deduplication should be UMI-aware rather than based on identical base pairs alone

## Inputs

- Paired-end FASTQ files, gzip-compressed (e.g. in.R1.fq.gz and in.R2.fq.gz)
- fastp JSON report with a duplication-rate field (reference comparison)

## Outputs

- Estimated paired-end duplication rate as a value in the range 0-1
- Per-key2 counts accumulated in counting array C
- fastp JSON report (fastp.json) containing the duplication-rate field

## How to apply

Load a paired-end FASTQ dataset (gzip-compressed read1/read2 files such as in.R1.fq.gz and in.R2.fq.gz). Compute key1 by packing the first 12 bases of read1 into an integer offset used to flag presence in a 2^24-entry duplication array D; compute key2 by packing the first 32 bases of read2 into a 64-bit long integer and increment the corresponding entry in counting array C only for pairs whose key1 position is already flagged. Treat a read as duplicated only when all of its base pairs are identical to another, then aggregate counts to estimate the PE duplication rate. Run fastp with duplication evaluation enabled (default mode, or --dedup with dup_calc_accuracy level 3) to obtain its reference duplication rate from the JSON report and compare. fastp additionally exploits the paired-end overlap: it can merge PE reads (-m/--merge) and correct mismatched base pairs in overlapped regions when one base is high quality and the other ultra low quality.

## Related tools

- **fastp** (Reference implementation of the array-based paired-end duplication evaluation; provides the duplication-rate value in its JSON report for comparison) — https://github.com/OpenGene/fastp
- **C++** (Implementation language used to re-implement the key1/key2 duplication arrays with fast integer packing)
- **Python** (Alternative language for scripting the duplication-key estimator over the FASTQ input)
- **fastplong** (fastp's counterpart for long reads; not applicable to this PE short-read skill) — https://github.com/OpenGene/fastplong

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz --json fastp.json
```

## Evaluation signals

- The fastp JSON output exists and contains a duplication-rate field whose value lies in the range 0-1
- The re-implemented estimator's duplication rate agrees with fastp's reported value within the expected hash-collision tolerance (~0.01% of reads)
- key1 offsets index into exactly 2^24 entries of duplication array D, derived from the first 12 bp of read1
- key2 values are packed into 64-bit long integers from the first 32 bp of read2, and counting array C is incremented only when the key1 position was already flagged
- Reads are counted as duplicated only when all of their base pairs are identical to another read pair

## Limitations

- Hash-collision tolerance: the array-based approximation can differ from an exact calculation by roughly 0.01% of reads
- Duplication is defined as all base pairs being identical, so reads differing at even one base (e.g. sequencing errors) are not counted as duplicates
- fastp targets short reads; for long reads (Nanopore, PacBio, Cyclone) the README directs users to fastplong

## Evidence
