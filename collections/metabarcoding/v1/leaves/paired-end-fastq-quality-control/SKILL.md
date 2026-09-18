---
name: paired-end-fastq-quality-control
description: Use when raw paired-end Illumina FASTQ (e.g. NextSeq PE150, NovaSeq)
  needs quality control, adapter removal, and filtering before downstream analysis,
  and speed or a single-pass all-in-one tool matters.
when_to_use_negative:
- Input is long-read data from Nanopore, PacBio, or Cyclone devices — use fastplong
  instead, as fastp is designed for short reads (Illumina, MGI)
- The task requires comparing residual adapter content across tools with a custom
  mismatch-tolerance sweep (e.g. scanning 33 bp adapter sequences with 0-5 allowed
  mismatches), which needs an external Python scanning step beyond fastp's reports
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: 'Primary all-in-one preprocessor: quality profiling, adapter trimming, quality/length
    filtering, sliding-window cutting, polyG/polyX trimming in one pass'
  repo: https://github.com/OpenGene/fastp
- name: fastplong
  role: Alternative for long-read (Nanopore, PacBio, Cyclone) preprocessing when input
    is not Illumina short reads
  repo: https://github.com/OpenGene/fastplong
- name: Trimmomatic
  role: Java-based comparison tool for sliding-window quality pruning and adapter
    clipping (ILLUMINACLIP + SLIDINGWINDOW:4:20) in benchmarking against fastp
- name: Cutadapt
  role: Comparison tool for adapter removal with a supplied 33 bp adapter sequence
    in the adapter-removal benchmark
merged_aliases:
- paired-end-fastq-quality-filtering
merged_alias_records:
- alias: paired-end-fastq-quality-filtering
  slug: paired-end-fastq-quality-filtering
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:4247627e16fef4a98b6779a8bb454430135f09631d9d7506dc92095c3df9eb36
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
  - AfterQC
  - Cutadapt
  - Java
  - Python
  - SOAPnuke
  - Trimmomatic
  - fastp
  - fastplong
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# paired-end-fastq-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

All-in-one FASTQ quality control and preprocessing for paired-end (and single-end) Illumina short reads using fastp, which performs quality profiling, adapter trimming, quality filtering, per-read sliding-window cutting, and optional polyG/polyX trimming in a single scan of the data.

## When to use

Use when raw paired-end Illumina FASTQ (e.g. NextSeq PE150, NovaSeq) needs quality control, adapter removal, and filtering before downstream analysis, and speed or a single-pass all-in-one tool matters.

## When NOT to use

- Input is long-read data from Nanopore, PacBio, or Cyclone devices — use fastplong instead, as fastp is designed for short reads (Illumina, MGI)
- The task requires comparing residual adapter content across tools with a custom mismatch-tolerance sweep (e.g. scanning 33 bp adapter sequences with 0-5 allowed mismatches), which needs an external Python scanning step beyond fastp's reports

## Inputs

- Paired-end FASTQ files (optionally gzip-compressed), e.g. in.R1.fq.gz and in.R2.fq.gz
- Single-end FASTQ files (compressed or uncompressed), e.g. in.fq

## Outputs

- Filtered/cleaned FASTQ output files (out.R1.fq.gz, out.R2.fq.gz; optionally unpaired reads and failed reads)
- fastp.html single-page QC report (before/after quality curves, Q20/Q30, adapter contents, duplication)
- fastp. machine-readable report for downstream interpretation

## How to apply

Run fastp on the paired FASTQ files (e.g. 'fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz'), letting default filters operate: quality filtering (qualified_quality_phred Q15, unqualified_percent_limit 40%, length_required 15, n_base_limit 5) and length filtering, both enabled by default. Enable --detect_adapter_for_pe so PE adapter sequences are automatically detected and trimmed without manual input, and consider per-read sliding-window cutting (like Trimmomatic but faster) to prune low-quality 5'/3' tails. For NovaSeq/NextSeq data, polyG in 3' ends is detected and trimmed; polyX trimming is off by default and enabled with -x/--trim_poly_x. Collect the JSON and single-page HTML reports, which profile quality curves, base contents, KMER, Q20/Q30, GC ratio, duplication, and adapter contents for both before- and after-filtering data. Rationale: fastp performs all these operations with a single scan of the FASTQ data and is 2-5 times faster than tools such as Trimmomatic or Cutadapt, making it suitable for large datasets like the 9316 M-base B17NCB1 benchmark.

## Related tools

- **fastp** (Primary all-in-one preprocessor: quality profiling, adapter trimming, quality/length filtering, sliding-window cutting, polyG/polyX trimming in one pass) — https://github.com/OpenGene/fastp
- **fastplong** (Alternative for long-read (Nanopore, PacBio, Cyclone) preprocessing when input is not Illumina short reads) — https://github.com/OpenGene/fastplong
- **Trimmomatic** (Java-based comparison tool for sliding-window quality pruning and adapter clipping (ILLUMINACLIP + SLIDINGWINDOW:4:20) in benchmarking against fastp)
- **Cutadapt** (Comparison tool for adapter removal with a supplied 33 bp adapter sequence in the adapter-removal benchmark)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz --detect_adapter_for_pe -w 1 -j fastp. -h fastp.html
```

## Evaluation signals

- fastp.html and fastp. reports exist and contain before/after quality profiles (Q20/Q30, GC ratio, duplication, adapter contents)
- Filtered reads pass the default thresholds: qualified_quality_phred Q15, unqualified_percent_limit 40%, length_required 15, n_base_limit 5
- Scanning filtered output for the 33 bp adapter sequence with up to 4 allowed mismatches yields no suspected adapters (as reported for fastp-filtered PE150 data)
- On the single-threaded B17NCB1 benchmark (9316 M bases), PE processing time is in the range reported for fastp (≈13.3 min, ≈116750 reads/s), i.e. 2-5x faster than Trimmomatic/Cutadapt (e.g. Trimmomatic PE 60.9 min)
- Read counts of R1 and R2 outputs remain paired and consistent after filtering

## Limitations

- polyG tail trimming is specific to NovaSeq/NextSeq two-color chemistry artifacts; polyX trimming is disabled by default and must be enabled explicitly with -x/--trim_poly_x
- Quality and length filtering are enabled by default and must be explicitly disabled (-Q, -L) if unfiltered output is desired; low complexity filter is disabled by default (-y to enable)
- Adapter trimming speed advantage holds for short reads; for long reads fastp is out of scope and fastplong should be used
- Comparison benchmarks are single-threaded (-w 1); multi-threading changes absolute timings

## Evidence
