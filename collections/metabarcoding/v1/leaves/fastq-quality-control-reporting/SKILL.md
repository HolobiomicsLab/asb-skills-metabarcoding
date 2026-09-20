---
name: fastq-quality-control-reporting
description: Use when Illumina (NovaSeq/MGI) FASTQ reads, especially paired-end, need
  QC profiling plus cleaning — adapter trimming, quality filtering, duplication-rate
  estimation — before downstream alignment or variant calling.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — the fastp README directs those
  to fastplong instead
- You need a standalone QC pass with no output FASTQ or trimming — a dedicated QC-only
  tool may suffice, though fastp can serve this role
- Duplication assessment requires strict per-mate analysis identical to FASTQC for
  comparability — fastp's PE-aware duplication rate is not directly comparable
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: 'Ultrafast all-in-one preprocessor: single-scan QC profiling, adapter trimming,
    quality/length/low-complexity filtering, PE duplication-rate evaluation, JSON/HTML
    report generation'
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Baseline per-mate QC tool whose single-end duplication evaluation (analyzing
    read1 and read2 individually) is contrasted against fastp's paired-end-aware rate
- name: Python
  role: High-level scripting for report parsing and batch/preprocessing drivers such
    as parallel.py
  repo: https://github.com/OpenGene/fastp/blob/master/parallel.py
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:2fb4231377cb76dd9e42ba2820c69a679b95b54ed3edc2a9b1a09167756825d8
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
  - Cutadapt
  - FASTQC
  - Python
  - SOAPnuke
  - SpeedSeq
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# fastq-quality-control-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profile and report FASTQ quality while preprocessing reads with fastp, which performs quality control, adapter trimming, quality filtering and per-read quality pruning in a single scan of the FASTQ data, and emits JSON/HTML reports (before/after filtering) that are faster and more informative than a standalone FASTQC pass.

## When to use

Use when Illumina (NovaSeq/MGI) FASTQ reads, especially paired-end, need QC profiling plus cleaning — adapter trimming, quality filtering, duplication-rate estimation — before downstream alignment or variant calling.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — the fastp README directs those to fastplong instead
- You need a standalone QC pass with no output FASTQ or trimming — a dedicated QC-only tool may suffice, though fastp can serve this role
- Duplication assessment requires strict per-mate analysis identical to FASTQC for comparability — fastp's PE-aware duplication rate is not directly comparable

## Inputs

- Single-end FASTQ file (optionally gzip-compressed)
- Paired-end FASTQ files: R1 and R2 mates
- Interleaved FASTQ input (supported)

## Outputs

- Filtered FASTQ files (R1/R2, optionally split as 0001.R1.gz, 0002.R1.gz...)
- fastp JSON report with before/after filtering statistics and duplication rates
- Single-page HTML quality-control report (quality curves, base contents, KMER, Q20/Q30, GC ratio, duplication, adapter contents)
- Unpaired reads / reads failing filters (if enabled)

## How to apply

Run fastp in paired-end mode (fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz) so adapter auto-detection, quality filtering (default qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15) and duplication-rate evaluation run in a single scan; capture the JSON report for machine parsing and the single HTML page for visual QC of quality curves, Q20/Q30, GC ratio and adapter/duplication content both before and after filtering. For duplication assessment on high-depth PE data (e.g. ~10000x cfDNA), prefer fastp's PE duplication rate over FASTQC, which analyzes read1 and read2 individually and overestimates duplication (FASTQC ~79.99%/77.75% vs fastp PE 16.22% in the paper's example). Validate by checking that the filtered output retains pairs, the JSON report contains both before/after statistics, and downstream variant calling on filtered data (e.g. SpeedSeq on NA12878 SRR952827) yields fewer false positives with fastp (6708) than Trimmomatic (7174), Cutadapt (6942) or SOAPnuke (7040).

## Related tools

- **fastp** (Ultrafast all-in-one preprocessor: single-scan QC profiling, adapter trimming, quality/length/low-complexity filtering, PE duplication-rate evaluation, JSON/HTML report generation) — https://github.com/OpenGene/fastp
- **FASTQC** (Baseline per-mate QC tool whose single-end duplication evaluation (analyzing read1 and read2 individually) is contrasted against fastp's paired-end-aware rate)
- **Python** (High-level scripting for report parsing and batch/preprocessing drivers such as parallel.py) — https://github.com/OpenGene/fastp/blob/master/parallel.py

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp. --html fastp.html
```

## Evaluation signals

- fastp JSON report exists and contains quality statistics for both before and after filtering data (Q20/Q30, GC ratio, duplication, adapter contents)
- On high-depth PE data (e.g. ~10000x cfDNA), the fastp PE duplication rate is markedly lower than FASTQC per-mate rates (reported ~16.22% vs ~79.99%/77.75% FASTQC, and ~79.23%/79.06% fastp single-end)
- Output R1/R2 FASTQ files remain paired (equal read counts) after filtering with length_required 15 and qualified_quality_phred 15 defaults
- Downstream validation: variant calling (e.g. SpeedSeq on NA12878 SRR952827) on fastp-filtered reads yields the fewest false positives (fastp 6708 vs Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174)
- HTML report renders a single page comparing pre- and post-filtering quality curves

## Limitations

- fastp is designed for short reads (Illumina NovaSeq, MGI); for Nanopore/PacBio/Cyclone long reads use fastplong instead
- FASTQC duplication evaluation lacks paired-end support and overestimates duplication on high-depth PE data; per-mate FASTQC numbers should not be compared head-to-head with fastp's PE rate
- Several features are disabled by default — low-complexity filter (-y), polyX trimming (-x), overrepresented-sequence analysis (-p) — so enable them explicitly when needed; quality and length filtering are enabled by default and must be disabled (-Q/-L) if unwanted

## Evidence
