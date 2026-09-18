---
name: single-end-vs-paired-end-mode-comparison
description: Use when assessing duplication in high-depth paired-end FASTQ data (e.g.
  ~10000x cfDNA) and FASTQC per-mate duplication rates appear inflated, requiring
  a paired-end-aware cross-check.
when_to_use_negative:
- Data are long reads (Nanopore, PacBio, Cyclone) — use fastplong instead of fastp
- Input is single-end data only, where no paired-end mode comparison is possible
- Duplication estimates are needed for downstream deduplication decisions on UMI-enabled
  data — fastp UMI processing (-U/--umi) should be considered instead
tools:
- name: fastp
  role: Runs in both paired-end mode (PE duplication rate) and single-end mode (per-mate
    SE duplication rate); emits JSON/HTML reports with duplication statistics
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Baseline duplication evaluation on read1 and read2 individually, without paired-end
    support, used to demonstrate PE overestimation
- name: Python
  role: Parses fastp JSON/HTML and FASTQC reports to extract duplication rates and
    assemble the comparison table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:b2ed653c986dd4c19d1496f06489ef9d5f355919bed7a10a7f4bed8df496b805
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-end-vs-paired-end-mode-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that quantifies how single-end duplication evaluation (FASTQC on each mate) overestimates duplication on high-depth paired-end cfDNA FASTQ data relative to fastp's paired-end-aware duplication analysis (~80% vs ~16% on ~10000x data).

## When to use

Use when assessing duplication in high-depth paired-end FASTQ data (e.g. ~10000x cfDNA) and FASTQC per-mate duplication rates appear inflated, requiring a paired-end-aware cross-check.

## When NOT to use

- Data are long reads (Nanopore, PacBio, Cyclone) — use fastplong instead of fastp
- Input is single-end data only, where no paired-end mode comparison is possible
- Duplication estimates are needed for downstream deduplication decisions on UMI-enabled data — fastp UMI processing (-U/--umi) should be considered instead

## Inputs

- Paired-end FASTQ files (e.g. gzipped in.R1.fq.gz and in.R2.fq.gz) from high-depth (~10000x) cell-free DNA sequencing
- Single-end FASTQ files extracted per mate for SE-mode comparison

## Outputs

- FASTQC per-mate duplication-rate values (e.g. 79.99% read1, 77.75% read2)
- fastp PE duplication rate from the fastp JSON/HTML report (e.g. 16.22%)
- fastp SE duplication rates per mate (e.g. 79.23% and 79.06%)
- Comparison table with columns: tool, mode (PE/SE), read mate, duplication rate

## How to apply

Run FASTQC on read1 and read2 separately to record per-mate duplication rates (reported: 79.99% and 77.75% on ~10000x cfDNA PE data). Then run fastp in paired-end mode (fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz) with default duplication-rate evaluation enabled and extract the PE duplication rate from the fastp JSON/HTML report (reported: 16.22%). Next, run fastp separately on each mate as single-end input (fastp -i in.fq -o out.fq) to obtain SE duplication rates (reported: ~79.23% and ~79.06%), which should agree with the FASTQC per-mate estimates. Assemble a comparison table (tool, mode, read mate, duplication rate) and confirm the fastp PE rate is markedly lower than both the FASTQC PE estimates and fastp's own SE rates; the rationale is that FASTQC analyzes read1 and read2 individually without paired-end support, counting mates of the same fragment as duplicates.

## Related tools

- **fastp** (Runs in both paired-end mode (PE duplication rate) and single-end mode (per-mate SE duplication rate); emits JSON/HTML reports with duplication statistics) — https://github.com/OpenGene/fastp
- **FASTQC** (Baseline duplication evaluation on read1 and read2 individually, without paired-end support, used to demonstrate PE overestimation)
- **Python** (Parses fastp JSON/HTML and FASTQC reports to extract duplication rates and assemble the comparison table)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -- fastp_pe.
```

## Evaluation signals

- fastp PE duplication rate (~16.22%) is markedly lower than both FASTQC PE estimates (~79.99%/77.75%) and fastp's own SE rates (~79.23%/79.06%)
- fastp SE duplication rates agree closely with FASTQC per-mate rates, isolating the PE-awareness of fastp as the source of the difference
- All duplication-rate values are successfully extracted from the fastp JSON/HTML reports and FASTQC reports with matching mate labels
- Comparison table rows cover all four tool/mode combinations (FASTQC PE per-mate, fastp PE, fastp SE per mate) with no missing values

## Limitations

- FASTQC analyzes read1 and read2 individually without paired-end support, systematically overestimating duplication for high-depth PE data
- fastp's default duplication-rate evaluation must be explicitly captured from reports; some related analyses such as overrepresented sequence analysis are disabled by default (-p/--overrepresentation_analysis to enable)
- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read data require fastplong

## Evidence
