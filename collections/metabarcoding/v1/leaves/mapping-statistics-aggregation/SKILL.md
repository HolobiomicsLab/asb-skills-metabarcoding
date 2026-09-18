---
name: mapping-statistics-aggregation
description: Use when comparing how well FASTQ reads filtered by different preprocessing
  tools align to a reference genome, and you need per-tool mapping metrics like mismatches,
  clips, and single-read maps.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp/BWA-MEM on Illumina-style
  short reads does not apply; use fastplong instead
- The question is about preprocessing speed or QC report generation rather than downstream
  alignment quality comparison
- Only one preprocessing tool is available and no raw-data baseline exists — a relative
  ranking cannot be computed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: produces one of the filtered FASTQ inputs whose alignment quality is evaluated
    (and empirically the best-performing filter)
  repo: https://github.com/OpenGene/fastp
- name: BWA
  role: BWA-MEM aligner used to map each filtered read pair set to hg19
- name: Samtools
  role: sort, index, and flagstat/stats evaluation of alignments to extract mismatched
    bases, clipped reads, and single-read maps
- name: Trimmomatic
  role: competing preprocessing tool whose filtered FASTQ output is compared against
    fastp's
- name: Cutadapt
  role: competing preprocessing tool whose filtered FASTQ output is compared against
    fastp's
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:4f2b9049e90a5df425cc04748fa46fe73f22fd574b8679526516a80b8d507828
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
  - BWA
  - Cutadapt
  - Samtools
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mapping-statistics-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates BWA-MEM/Samtools alignment statistics (mapped bases, mismatched bases, soft/hard-clipped reads, single-read maps) for NS_PE150 FASTQ data preprocessed by different tools, to rank preprocessing quality against a raw-data baseline.

## When to use

Use when comparing how well FASTQ reads filtered by different preprocessing tools align to a reference genome, and you need per-tool mapping metrics like mismatches, clips, and single-read maps.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp/BWA-MEM on Illumina-style short reads does not apply; use fastplong instead
- The question is about preprocessing speed or QC report generation rather than downstream alignment quality comparison
- Only one preprocessing tool is available and no raw-data baseline exists — a relative ranking cannot be computed

## Inputs

- gzip-compressed paired-end filtered FASTQ files (in.R1.fq.gz / in.R2.fq.gz style outputs) from each preprocessing tool
- raw unfiltered NS_PE150 paired-end FASTQ reads (baseline)
- hg19 reference genome for BWA-MEM alignment

## Outputs

- per-tool mapping metrics table (table2_mapping_stats.tsv): mapped bases, mismatched bases, clip reads, single-read maps
- sorted, indexed BWA-MEM BAM files per input read set
- Samtools flagstat/stats reports per alignment

## How to apply

Collect the gzip-compressed paired-end filtered FASTQ outputs (in.R1.fq.gz / in.R2.fq.gz style) from each preprocessing tool (fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt) plus the unfiltered raw NS_PE150 reads. Align each read pair set to hg19 with BWA-MEM, then sort, index, and evaluate the BAMs with Samtools flagstat/stats to extract mapped bases, mismatched bases, clipped reads, and reads whose mate failed to map (single-read maps). Use the raw-data alignment (19.8 M mismatched bases) as the unprocessed baseline and tabulate metrics per tool. Validation: confirm the fastp-preprocessed alignment reproduces Table 2 values (3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps) within tolerance, and that fastp's mismatch count is substantially lower than raw data's, then write table2_mapping_stats.tsv.

## Related tools

- **fastp** (produces one of the filtered FASTQ inputs whose alignment quality is evaluated (and empirically the best-performing filter)) — https://github.com/OpenGene/fastp
- **BWA** (BWA-MEM aligner used to map each filtered read pair set to hg19)
- **Samtools** (sort, index, and flagstat/stats evaluation of alignments to extract mismatched bases, clipped reads, and single-read maps)
- **Trimmomatic** (competing preprocessing tool whose filtered FASTQ output is compared against fastp's)
- **Cutadapt** (competing preprocessing tool whose filtered FASTQ output is compared against fastp's)

## Examples

```
bwa mem hg19.fa in.R1.fq.gz in.R2.fq.gz | samtools sort -o aln.bam; samtools index aln.bam; samtools stats aln.bam > aln.stats
```

## Evaluation signals

- fastp-preprocessed alignment reproduces Table 2 values: 3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps
- fastp mismatched base count is substantially lower than the raw-data baseline of 19.8 M mismatched bases
- fastp shows the lowest values across all three metrics (mismatches, clips, single-read maps) versus AfterQC, SOAPnuke, Trimmomatic, Cutadapt, and raw
- table2_mapping_stats.tsv contains one row per preprocessing tool including the raw baseline
- single-read map counts fall in tens (71 for fastp), so mate-pair coherence is near-perfect after filtering

## Limitations

- Conclusions depend on the specific NS_PE150 dataset and hg19 reference; other read sets may rank tools differently
- The comparison conflates filter aggressiveness with alignment quality — tools that remove more reads reduce mismatches but also reduce total mapped bases
- Samtools flagstat/stats metrics (clips, single-read maps) are proxy indicators of cleanliness, not direct measures of biological correctness

## Evidence

- [other] workflow: [evidence span withheld — anchor status: not_found]
- [results] finding: [evidence span withheld — anchor status: not_found]
