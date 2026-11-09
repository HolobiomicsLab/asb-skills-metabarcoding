---
name: paired-end-overlap-analysis
description: Use when assessing duplication levels of high-depth (e.g. ~10000x cfDNA)
  paired-end Illumina FASTQ data, or when deciding whether a tool's duplication metric
  accounts for read1/read2 pairing.
when_to_use_negative:
- Input is true single-end data — the pairing-aware analysis and the PE-vs-SE contrast
  have no meaning
- Low-depth sequencing data where duplication rates are negligible and the PE overestimation
  effect is immaterial
- Long-read (Nanopore/PacBio) data — fastp is designed for short reads; fastplong
  is the recommended tool instead
edam_operation: http://edamontology.org/operation_0237
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: Runs paired-end duplication-rate evaluation (PE mode with -i/-I inputs) and
    single-end evaluation per mate; duplication rate is enabled by default and reported
    in JSON/HTML
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Baseline per-mate duplication-rate reporter that evaluates read1 and read2
    individually without paired-end support, producing the inflated estimates under
    comparison
- name: Python
  role: Extract duplication-rate values from fastp JSON reports and assemble the comparison
    table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:4b484da3a13b33e3c2be778f8cf5c48f1b4a298babef3aecbd3aeded4f72b293
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
  edam_operation: http://edamontology.org/operation_0237
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# paired-end-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for evaluating duplication rates in high-depth paired-end FASTQ data, contrasting FASTQC's per-mate (single-end) duplication estimates with fastp's paired-end-aware duplication evaluation to expose PE overestimation.

## When to use

Use when assessing duplication levels of high-depth (e.g. ~10000x cfDNA) paired-end Illumina FASTQ data, or when deciding whether a tool's duplication metric accounts for read1/read2 pairing.

## When NOT to use

- Input is true single-end data — the pairing-aware analysis and the PE-vs-SE contrast have no meaning
- Low-depth sequencing data where duplication rates are negligible and the PE overestimation effect is immaterial
- Long-read (Nanopore/PacBio) data — fastp is designed for short reads; fastplong is the recommended tool instead

## Inputs

- Paired-end FASTQ files (in.R1.fq.gz and in.R2.fq.gz) from high-depth cfDNA sequencing
- fastp report files (JSON/HTML) with duplication-rate evaluation enabled
- FASTQC per-mate reports for read1 and read2

## Outputs

- Per-mate duplication rates from FASTQC (e.g. read1 and read2 percentages)
- fastp PE duplication rate from the fastp report
- fastp SE duplication rates from single-end fastp runs on each mate
- Comparison table (tool, mode, read mate, duplication rate)

## How to apply

Run FASTQC separately on read1 and read2 of the paired-end cell-free DNA FASTQ files and record the per-mate duplication rates (FASTQC analyzes each mate individually without paired-end support). Then run fastp in paired-end mode with default duplication-rate evaluation enabled and capture the PE duplication rate from the fastp report. As a control, also run fastp on each mate independently as single-end input (fastp -i in.fq -o out.fq) to obtain SE duplication rates, showing that the discrepancy comes from pairing-awareness rather than the tool per se. Extract duplication-rate values from the fastp JSON/HTML reports and FASTQC reports, then assemble a comparison table (tool, mode, read mate, duplication rate). The rationale is that single-end evaluation counts both mates of one duplicated fragment as duplicates, inflating the rate; fastp's PE-aware analysis of the paired fragment reports the true duplication level.

## Related tools

- **fastp** (Runs paired-end duplication-rate evaluation (PE mode with -i/-I inputs) and single-end evaluation per mate; duplication rate is enabled by default and reported in JSON/HTML) — https://github.com/OpenGene/fastp
- **FASTQC** (Baseline per-mate duplication-rate reporter that evaluates read1 and read2 individually without paired-end support, producing the inflated estimates under comparison)
- **Python** (Extract duplication-rate values from fastp JSON reports and assemble the comparison table)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp. --html fastp.html
```

## Evaluation signals

- fastp PE duplication rate (~16.22%) is markedly lower than both FASTQC per-mate estimates (~79.99% read1, ~77.75% read2) and fastp's own SE rates (~79.23% read1, ~79.06% read2)
- fastp SE rates on each mate (~79%) are consistent with FASTQC's per-mate rates, confirming the inflation is due to lack of pairing-awareness, not tool implementation differences
- All duplication rates are successfully extracted from fastp JSON/HTML reports and FASTQC reports
- Comparison table contains a row for each tool/mode/mate combination with a recorded percentage

## Limitations

- FASTQC analyzes read1 and read2 individually without paired-end support, so its duplication metric systematically overestimates duplication for PE data
- fastp's overrepresented sequence analysis and other QC features are disabled by default and must be explicitly enabled (e.g. -p), whereas duplication-rate evaluation is on by default
- fastp is designed for short reads (Illumina NovaSeq, MGI); for long reads use fastplong instead

## Evidence

- [results] fastp reported only 16.22% PE duplication vs FASTQC's 79.99%/77.75% per-mate rates: [evidence span withheld — anchor status: not_found]
