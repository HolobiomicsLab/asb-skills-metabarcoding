---
name: variant-calling-from-aligned-reads
description: Use when comparing preprocessing tools by their effect on variant-calling
  false positives, or when preprocessing paired-end Illumina FASTQ reads (e.g. SRR952827)
  before alignment and variant calling.
when_to_use_negative:
- Input reads are long reads (Nanopore, PacBio) — fastp is designed for short reads
  and the authors direct long-read users to fastplong
- The goal is only QC reporting, not a controlled tool-to-tool comparison — a single
  fastp run with its HTML/JSON report suffices
- No curated variant truth set is available, so false positives cannot be counted
  and the comparison is not meaningful
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Preprocesses raw paired-end FASTQ with automatic adapter detection and default
    quality filtering before alignment and variant calling
  repo: https://github.com/OpenGene/fastp
- name: SpeedSeq
  role: Aligns each preprocessed read pair to the reference genome and calls variants
    for false-positive counting
- name: Trimmomatic
  role: Comparator preprocessor applying LEADING/TRAILING/SLIDINGWINDOW-style trimming
    to the same raw pairs
- name: Cutadapt
  role: Comparator preprocessor performing adapter trimming on the same raw pairs
- name: SOAPnuke
  role: Comparator preprocessor with comparable adapter-trimming and quality-pruning
    settings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:ccac5d1df2413f0c3dd3ca3931494f7fbe6f29d42a6a9129825528efc4762a1a
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
  - SOAPnuke
  - SpeedSeq
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# variant-calling-from-aligned-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark how FASTQ preprocessing tool choice (fastp vs Trimmomatic, Cutadapt, SOAPnuke) affects downstream variant-calling specificity, by aligning each preprocessed read pair and counting false positive variants against a curated truth set (e.g. NA12878).

## When to use

Use when comparing preprocessing tools by their effect on variant-calling false positives, or when preprocessing paired-end Illumina FASTQ reads (e.g. SRR952827) before alignment and variant calling.

## When NOT to use

- Input reads are long reads (Nanopore, PacBio) — fastp is designed for short reads and the authors direct long-read users to fastplong
- The goal is only QC reporting, not a controlled tool-to-tool comparison — a single fastp run with its HTML/JSON report suffices
- No curated variant truth set is available, so false positives cannot be counted and the comparison is not meaningful

## Inputs

- Paired-end FASTQ reads (e.g. SRR952827 for NA12878)
- Reference genome FASTA
- Curated variant truth set (e.g. NA12878 curated variants)
- Preprocessing tools: fastp, Trimmomatic, Cutadapt, SOAPnuke

## Outputs

- Preprocessed paired-end FASTQ files per preprocessing tool
- Per-tool variant callsets (SpeedSeq)
- fp_counts.csv with false-positive variant counts per preprocessor

## How to apply

Download the paired-end FASTQ reads (e.g. NA12878 accession SRR952827), a curated variant truth set, and the reference genome. Preprocess the raw pairs with fastp in paired-end mode (`fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz`) using default adapter auto-detection and default quality filtering (qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15), then preprocess the same pairs with Cutadapt, SOAPnuke, and Trimmomatic using comparable adapter-trimming and sliding-window quality-pruning settings. For each of the four preprocessed read pairs, align to the reference and call variants with SpeedSeq. Intersect each tool's callset with the curated variants and count false positives (called variants absent from the truth set); the reported expectation is fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174 false positives, demonstrating fastp improves downstream specificity.

## Related tools

- **fastp** (Preprocesses raw paired-end FASTQ with automatic adapter detection and default quality filtering before alignment and variant calling) — https://github.com/OpenGene/fastp
- **SpeedSeq** (Aligns each preprocessed read pair to the reference genome and calls variants for false-positive counting)
- **Trimmomatic** (Comparator preprocessor applying LEADING/TRAILING/SLIDINGWINDOW-style trimming to the same raw pairs)
- **Cutadapt** (Comparator preprocessor performing adapter trimming on the same raw pairs)
- **SOAPnuke** (Comparator preprocessor with comparable adapter-trimming and quality-pruning settings)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- Reproduced false-positive counts match the reported values: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174
- fp_counts.csv exists and contains exactly four rows, one per preprocessor, with the expected ordering (fastp lowest)
- Each preprocessed read pair is paired (R1/R2 read counts remain consistent per tool)
- Each per-tool callset intersects cleanly with the curated NA12878 truth set so FP = called variants minus truth-set members

## Limitations

- fastp's default quality filtering, length filtering, and adapter auto-detection mean tool comparisons must ensure the comparator settings (e.g. Trimmomatic LEADING/TRAILING/SLIDINGWINDOW equivalents) are actually comparable, or counts are not attributable
- fastp is built for short reads (Illumina NovaSeq, MGI); long-read data requires fastplong
- Some fastp filters (low complexity filter, polyX trimming, overrepresented sequence analysis) are disabled by default and must be explicitly enabled if desired

## Evidence
