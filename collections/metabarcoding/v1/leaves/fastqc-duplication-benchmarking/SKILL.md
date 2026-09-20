---
name: fastqc-duplication-benchmarking
description: Use when you need to assess duplication in high-depth paired-end Illumina
  FASTQ data (e.g. ~10000x cfDNA) and must choose between FASTQC per-mate estimates
  and fastp's paired-end duplication evaluation.
when_to_use_negative:
- Input is single-end FASTQ data, where the paired-end overestimation issue motivating
  this benchmark does not apply
- Data was already deduplicated or processed with UMI-based deduplication, making
  mate-pair duplication comparison redundant
- You need a general quality-control profile (Q20/Q30, GC, KMER, adapters) rather
  than a duplication-rate benchmark between tools
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Runs paired-end and single-end duplication-rate evaluation; emits duplication
    rates in its JSON/HTML reports
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Baseline duplication estimator that analyzes read1 and read2 individually
    without paired-end support
- name: Python
  role: Extraction of duplication-rate values from fastp JSON/HTML and FASTQC reports
    and assembly of the comparison table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:bf7402d319d0791741484c06a099bd391f0a269c2696e550f5b7fbe27099f167
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
  - FASTQC
  - Python
  - fastp
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fastqc-duplication-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for comparing duplication-rate estimates between FASTQC (single-read, mate-by-mate) and fastp (paired-end aware) on high-depth paired-end FASTQ data, demonstrating FASTQC's PE overestimation.

## When to use

Use when you need to assess duplication in high-depth paired-end Illumina FASTQ data (e.g. ~10000x cfDNA) and must choose between FASTQC per-mate estimates and fastp's paired-end duplication evaluation.

## When NOT to use

- Input is single-end FASTQ data, where the paired-end overestimation issue motivating this benchmark does not apply
- Data was already deduplicated or processed with UMI-based deduplication, making mate-pair duplication comparison redundant
- You need a general quality-control profile (Q20/Q30, GC, KMER, adapters) rather than a duplication-rate benchmark between tools

## Inputs

- Paired-end FASTQ files (in.R1.fq.gz, in.R2.fq.gz) from high-depth sequencing, e.g. ~10000x cell-free DNA

## Outputs

- FASTQC per-mate duplication rates (read1 and read2)
- fastp paired-end duplication rate from fastp. / fastp.html report
- fastp single-end duplication rates for read1 and read2 run individually
- Comparison table (tool, mode, read mate, duplication rate)

## How to apply

Run FASTQC on read1 and read2 of the paired-end cell-free DNA FASTQ files separately and record the per-mate duplication rates (in the reference case 79.99% for read1 and 77.75% for read2), since FASTQC analyzes each mate individually without paired-end support. Then run fastp in paired-end mode (fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz) with default duplication-rate evaluation enabled and capture the PE duplication rate from the generated report (reference value 16.22%). For a complete comparison, also run fastp separately on each mate as single-end input (fastp -i in.fq -o out.fq) to obtain SE duplication rates (~79.23%/79.06% in the reference case). Extract all duplication values from the FASTQC reports and the fastp JSON/HTML reports, and assemble a comparison table by tool, mode, and read mate. The rationale is that duplication should be evaluated over read pairs, so fastp's PE-aware estimate (~16.22%) is the biologically meaningful one, while FASTQC and fastp-SE rates conflate mate-level duplication and inflate the apparent duplication level.

## Related tools

- **fastp** (Runs paired-end and single-end duplication-rate evaluation; emits duplication rates in its JSON/HTML reports) — https://github.com/OpenGene/fastp
- **FASTQC** (Baseline duplication estimator that analyzes read1 and read2 individually without paired-end support)
- **Python** (Extraction of duplication-rate values from fastp JSON/HTML and FASTQC reports and assembly of the comparison table)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp. --html fastp.html
```

## Evaluation signals

- fastp PE duplication rate (~16.22% on the reference ~10000x cfDNA dataset) is markedly lower than both FASTQC per-mate estimates (~79.99%/77.75%) and fastp's own SE rates (~79.23%/79.06%)
- fastp JSON report contains a duplication-rate entry for the PE run with values consistent with the HTML report
- FASTQC report duplication values for read1 and read2 are close to each other and to fastp SE rates, confirming mate-level (not pair-level) estimation
- All four rate values populate the comparison table with tool, mode, read mate, and rate columns filled

## Limitations

- FASTQC lacks paired-end support and analyzes each mate individually, so its duplication estimate is inflated for PE data and should not be used as the definitive duplication level for high-depth PE sequencing
- fastp is designed for short reads (Illumina NovaSeq, MGI); for long reads (Nanopore, PacBio) the analogous tool is fastplong, not fastp
- The demonstrated figures come from one ~10000x cfDNA dataset; the magnitude of FASTQC overestimation may differ at other depths or library types

## Evidence
