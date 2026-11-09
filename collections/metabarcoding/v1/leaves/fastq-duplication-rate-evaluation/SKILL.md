---
name: fastq-duplication-rate-evaluation
description: Use when estimating or comparing duplication rates on high-depth paired-end
  FASTQ data (e.g. ~10000x cfDNA), or when deciding whether FASTQC's per-mate duplication
  output overestimates the true PE duplication level.
when_to_use_negative:
- Input is long-read Nanopore/PacBio data — the README states fastp is designed for
  short reads (Illumina NovaSeq, MGI) and points long-read users to fastplong
- The goal is alignment-based duplicate marking on a BAM (e.g. after mapping), not
  FASTQ-level duplication rate evaluation
- Data are low-depth single-end reads, where the PE overestimation effect this skill
  demonstrates does not apply
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: fastp
  role: Paired-end FASTQ preprocessor; runs default duplication-rate evaluation in
    PE mode (and SE mode for controls) and emits the duplication rate in its JSON/HTML
    report
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Baseline per-mate duplication evaluation on read1 and read2 individually (no
    paired-end support), used to show the overestimation
- name: Python
  role: Extract duplication-rate values from fastp JSON/HTML and FASTQC reports and
    assemble the comparison table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:b17f2e0d3c124e62039f8bb343d45756131589f608a88cbe80a3f94a6136fcc8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fastq-duplication-rate-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate duplication rates on paired-end FASTQ data and compare per-mate tools (FASTQC) against paired-end-aware tools (fastp), which the paper shows avoids the overestimation FASTQC produces on high-depth PE data such as ~10000x cell-free DNA sequencing.

## When to use

Use when estimating or comparing duplication rates on high-depth paired-end FASTQ data (e.g. ~10000x cfDNA), or when deciding whether FASTQC's per-mate duplication output overestimates the true PE duplication level.

## When NOT to use

- Input is long-read Nanopore/PacBio data — the README states fastp is designed for short reads (Illumina NovaSeq, MGI) and points long-read users to fastplong
- The goal is alignment-based duplicate marking on a BAM (e.g. after mapping), not FASTQ-level duplication rate evaluation
- Data are low-depth single-end reads, where the PE overestimation effect this skill demonstrates does not apply

## Inputs

- Paired-end FASTQ files (e.g. gzipped in.R1.fq.gz / in.R2.fq.gz) from high-depth sequencing such as ~10000x cfDNA
- FASTQ data for optional single-end control runs per mate

## Outputs

- FASTQC per-mate duplication rates (one value per read mate)
- fastp PE duplication rate from the fastp JSON/HTML report
- fastp SE duplication rates from per-mate single-end runs
- Comparison table (tool, mode, read mate, duplication rate)

## How to apply

Run FASTQC independently on read1 and read2 of the paired-end FASTQ files and record the per-mate duplication rates (in the cfDNA case: 79.99% for read1 and 77.75% for read2). Then run fastp in paired-end mode with default duplication-rate evaluation enabled (fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz) and capture the PE duplication rate from the fastp JSON/HTML report (reported: 16.22%). To isolate why the estimates diverge, also run fastp separately on each mate as single-end inputs (fastp -i in.fq -o out.fq) and record the SE duplication rates (79.23% for read1, 79.06% for read2). The rationale is that FASTQC evaluates read1 and read2 individually without paired-end support, so on high-depth PE data it counts the same duplicated fragment twice per mate, whereas fastp's PE-aware evaluation collapses mates. Assemble a comparison table (tool, mode, read mate, duplication rate) to demonstrate the inflation.

## Related tools

- **fastp** (Paired-end FASTQ preprocessor; runs default duplication-rate evaluation in PE mode (and SE mode for controls) and emits the duplication rate in its JSON/HTML report) — https://github.com/OpenGene/fastp
- **FASTQC** (Baseline per-mate duplication evaluation on read1 and read2 individually (no paired-end support), used to show the overestimation)
- **Python** (Extract duplication-rate values from fastp JSON/HTML and FASTQC reports and assemble the comparison table)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- fastp PE duplication rate (~16.22%) is markedly lower than both FASTQC per-mate estimates (~79.99%/77.75%) and fastp's own SE rates (~79.23%/79.06%)
- fastp's own SE duplication rates on read1/read2 land near the FASTQC per-mate values, confirming the divergence comes from mate-pair awareness, not the tool's counting rule
- The fastp JSON/HTML report contains a duplication-rate field for the PE run and for each SE control run
- The assembled table has one row per (tool, mode, read mate) combination with a populated duplication rate

## Limitations

- FASTQC analyzes read1 and read2 individually without paired-end support, so its duplication output overestimates duplication on high-depth paired-end data
- fastp's duplication evaluation is FASTQ-level (sequence-based), not alignment-based, so it reflects exact/near-exact read duplication rather than mapped duplicates
- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read data require fastplong

## Evidence
