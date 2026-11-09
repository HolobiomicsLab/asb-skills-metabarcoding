---
name: variant-comparison-against-truth-set
description: Use when a variant callset (VCF from SpeedSeq or similar) must be scored
  against a curated NA12878 truth set to count false positives, or when comparing
  how different FASTQ preprocessors affect downstream variant-calling specificity.
when_to_use_negative:
- Input is raw FASTQ data with no variant callset yet — preprocessing/benchmarking
  of the FASTQ itself (fastp's core scope) rather than downstream variant scoring
  is needed
- No curated truth set exists for the sample, so false positives cannot be defined
  by set-difference
- Reads are long reads (Nanopore/PacBio/Cyclone) — the fastp preprocessing upstream
  of this skill targets short reads (Illumina NovaSeq, MGI); use fastplong instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: fastp
  role: FASTQ preprocessor whose default filtering (adapter auto-detection, qualified_quality_phred
    15, unqualified_percent_limit 40, length_required 15) is benchmarked by the downstream
    false-positive counts
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Comparison preprocessor (adapter trimming, sliding-window quality pruning)
    whose callset false positives are counted against the truth set
- name: Cutadapt
  role: Comparison preprocessor with adapter-trimming settings whose callset false
    positives are counted against the truth set
- name: SOAPnuke
  role: Comparison preprocessor whose callset false positives are counted against
    the truth set
- name: SpeedSeq
  role: Aligns each preprocessed read pair to the reference and calls variants for
    downstream intersection with the truth set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:425db60865b5ef2b1b709f27790e99dda5697db803054fd47a92e1d4c0c9c0d8
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
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# variant-comparison-against-truth-set

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies false-positive variants from a downstream callset (e.g. SpeedSeq on NA12878 SRR952827) by intersecting it with a curated truth set and counting called variants absent from the curated set, enabling benchmarking of upstream preprocessing choices like fastp vs Trimmomatic, Cutadapt, and SOAPnuke.

## When to use

Use when a variant callset (VCF from SpeedSeq or similar) must be scored against a curated NA12878 truth set to count false positives, or when comparing how different FASTQ preprocessors affect downstream variant-calling specificity.

## When NOT to use

- Input is raw FASTQ data with no variant callset yet — preprocessing/benchmarking of the FASTQ itself (fastp's core scope) rather than downstream variant scoring is needed
- No curated truth set exists for the sample, so false positives cannot be defined by set-difference
- Reads are long reads (Nanopore/PacBio/Cyclone) — the fastp preprocessing upstream of this skill targets short reads (Illumina NovaSeq, MGI); use fastplong instead

## Inputs

- Paired-end FASTQ reads for NA12878 accession SRR952827 (e.g. in.R1.fq.gz / in.R2.fq.gz)
- NA12878 curated variant truth set
- Reference genome
- Preprocessed FASTQ pairs from each tool under test (fastp, Trimmomatic, Cutadapt, SOAPnuke)

## Outputs

- Per-preprocessor false-positive variant counts (fp_counts.csv: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174)
- Variant callsets per preprocessor produced by SpeedSeq

## How to apply

Download the NA12878 paired-end FASTQ reads for accession SRR952827, the NA12878 curated variant truth set, and the reference genome. Preprocess each read pair with the preprocessor under test (e.g. fastp in paired-end mode with default adapter auto-detection and default quality filtering: qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15; or comparable adapter-trimming/sliding-window settings for Trimmomatic, Cutadapt, and SOAPnuke). For each preprocessed pair, align to the reference and call variants with SpeedSeq. Intersect each callset with the curated truth set and count false positives as called variants absent from the curated set; write the counts (fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174) to fp_counts.csv. The preprocessor yielding the fewest false positives demonstrates the best downstream specificity.

## Related tools

- **fastp** (FASTQ preprocessor whose default filtering (adapter auto-detection, qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15) is benchmarked by the downstream false-positive counts) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Comparison preprocessor (adapter trimming, sliding-window quality pruning) whose callset false positives are counted against the truth set)
- **Cutadapt** (Comparison preprocessor with adapter-trimming settings whose callset false positives are counted against the truth set)
- **SOAPnuke** (Comparison preprocessor whose callset false positives are counted against the truth set)
- **SpeedSeq** (Aligns each preprocessed read pair to the reference and calls variants for downstream intersection with the truth set)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- Reproduced false-positive counts match the reported values: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174
- fp_counts.csv contains exactly four tool entries with integer false-positive counts
- Each callset was produced from the same raw SRR952827 read pair after preprocessing with the corresponding tool, keeping alignment/calling (SpeedSeq) identical across arms
- Fastp's count is the minimum across the four preprocessors, confirming improved downstream specificity
- fastp's HTML/JSON quality reports confirm default filters were active (qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15)

## Limitations

- fastp is designed for short reads (Illumina NovaSeq, MGI); for long reads (Nanopore, PacBio, Cyclone) use fastplong
- False-positive counts depend on preprocessor parameter choices; comparable adapter-trimming and sliding-window quality-pruning settings must be used across tools for a fair comparison
- Quality filtering and length filtering are enabled by default in fastp and must be explicitly disabled (-Q / -L) if raw-pair baselines are desired

## Evidence

- [other] Filtering NA12878 with each tool and calling variants with SpeedSeq showed fastp yielded the fewest false positive variants: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174: [evidence span withheld — anchor status: not_found]
