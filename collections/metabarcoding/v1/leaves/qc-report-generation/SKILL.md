---
name: qc-report-generation
description: Use when paired-end FASTQ data needs a comprehensive before/after QC
  report whose duplication-rate metric must be computed or reproduced using fastp's
  array-based key1/key2 counting instead of a hash-table approach.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp is designed for short
  reads (Illumina NovaSeq, MGI); use fastplong instead
- Input is already a processed QC artifact (e.g. an existing JSON/HTML report or a
  feature table) rather than raw FASTQ reads
- An exact, collision-free per-read duplication census is required — the array-based
  scheme tolerates hash collisions (~0.01% of reads)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: fastp
  role: ultra-fast FASTQ preprocessor that scans the data once and produces the JSON/HTML
    QC report including the duplication-rate metric
  repo: https://github.com/OpenGene/fastp
- name: fastplong
  role: long-read counterpart (Nanopore/PacBio) referenced when fastp's QC reporting
    is out of scope
  repo: https://github.com/OpenGene/fastplong
- name: Python
  role: used to implement the key1/key2 duplication estimator and compare against
    fastp's reported rate
- name: C++
  role: implementation language of fastp; also a vehicle for the array-based duplication
    estimator
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:b6c8a43a9abb8782192fea7d352d6075b24a82749a2ff4660beb49aceb57f150
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# -qc-report-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates fastp-style FASTQ quality-control reports (JSON + single-page HTML) that profile Q20/Q30, KMER, GC ratio, adapter content and — critically — a duplication-rate metric computed with fastp's array-based paired-end key1/key2 scheme rather than FASTQC's hash tables.

## When to use

Use when paired-end FASTQ data needs a comprehensive before/after QC report whose duplication-rate metric must be computed or reproduced using fastp's array-based key1/key2 counting instead of a hash-table approach.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp is designed for short reads (Illumina NovaSeq, MGI); use fastplong instead
- Input is already a processed QC artifact (e.g. an existing JSON/HTML report or a feature table) rather than raw FASTQ reads
- An exact, collision-free per-read duplication census is required — the array-based scheme tolerates hash collisions (~0.01% of reads)

## Inputs

- gzip-compressed paired-end FASTQ files (e.g. in.R1.fq.gz and in.R2.fq.gz)
- single-end (uncompressed) FASTQ files (e.g. in.fq)
- optionally a Python or C++ script implementing the key1/key2 duplication estimator

## Outputs

- fastp JSON report containing a duplication-rate field (value in 0-1) plus Q20/Q30, KMER, GC ratio and adapter-content metrics
- single HTML page visualizing quality control and filtering results before and after filtering
- filtered FASTQ outputs from the preprocessing run

## How to apply

Run fastp over paired-end FASTQ (e.g. in.R1.fq.gz / in.R2.fq.gz) with duplication evaluation enabled (default mode, or --dedup with dup_calc_accuracy level 3) so it emits a JSON report and a single HTML page profiling both before- and after-filtering data. To reproduce the PE duplication metric, pack the first 12 bases of each read1 into key1, used as an offset flagging presence in a 2^24-entry duplication array D, and pack the first 32 bases of each read2 into a 64-bit key2; for pairs whose key1 offset is already flagged, increment the entry in counting array C for key2. Aggregate the counts to estimate duplication rate, treating a read as duplicated only when all of its base pairs are identical to another. Validate the report by checking that fastp. contains a duplication-rate field in [0,1] that agrees with fastp's own calculation within the expected hash-collision tolerance (~0.01% of reads).

## Related tools

- **fastp** (ultra-fast FASTQ preprocessor that scans the data once and produces the JSON/HTML QC report including the duplication-rate metric) — https://github.com/OpenGene/fastp
- **fastplong** (long-read counterpart (Nanopore/PacBio) referenced when fastp's QC reporting is out of scope) — https://github.com/OpenGene/fastplong
- **Python** (used to implement the key1/key2 duplication estimator and compare against fastp's reported rate)
- **C++** (implementation language of fastp; also a vehicle for the array-based duplication estimator) — https://github.com/OpenGene/fastp

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp. --html fastp.html
```

## Evaluation signals

- The output report fastp. exists and contains a duplication-rate field whose value lies in the range 0-1
- The implemented key1/key2 estimator's duplication rate agrees with fastp's reference value within the expected hash-collision tolerance (~0.01% of reads)
- The HTML report renders quality curves, base contents, KMER, Q20/Q30, GC ratio and duplication for both before- and after-filtering data
- The report covers a single scan of the FASTQ data — quality control, adapter trimming and filtering statistics appear together without re-reading input
- key1 offsets index a 2^24-entry array D and key2 is a 64-bit integer into counting array C, matching the paper's scheme

## Limitations

- The array-based PE duplication evaluation permits hash collisions, introducing ~0.01% of reads of error versus an exact method
- A read is treated as duplicated only when all of its base pairs are identical to another, so partially-overlapping duplicates are not counted
- fastp targets short reads (Illumina NovaSeq, MGI); for Nanopore/PacBio/Cyclone data the README directs users to fastplong

## Evidence
