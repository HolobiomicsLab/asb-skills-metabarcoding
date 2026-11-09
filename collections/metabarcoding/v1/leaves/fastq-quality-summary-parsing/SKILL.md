---
name: fastq-quality-summary-parsing
description: Use when raw paired-end Illumina FASTQ reads need DADA2/Deblur truncation
  lengths derived from per-base quality profiles rather than manual inspection or
  fixed defaults.
when_to_use_negative:
- DADA2/Deblur truncation lengths are already fixed in the config and no quality-based
  re-estimation is desired
- Input is already a denoised feature table or representative sequences rather than
  raw FASTQ with per-base quality
- Data are single-end or non-Illumina reads whose quality profiles do not map to the
  Read1/Read2 dropoff logic
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Tourmaline
  role: Source of the fastqc_per_base_sequence_quality_dropoff.py helper and the config
    truncation parameters (dada2_trunc_len_f, dada2pe_trunc_len_r)
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Hosts the DADA2/Deblur denoising that consumes the suggested truncation lengths
- name: DADA2
  role: Denoising algorithm whose trunc_len_f/trunc_len_r parameters are informed
    by the parsed quality dropoff
  repo: github.com/benjjneb/dada2
- name: Snakemake
  role: Workflow system in which the qaqc step and trimming-parameter determination
    are executed
- name: Python
  role: Language used to reimplement the dropoff estimator that scans per-base median
    quality
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:9851152f7d5311a7ae55c1f18e0d3c68c5638f448284dcb4728396cd5dc682a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cutadapt
  - DADA2
  - Python
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fastq-quality-summary-parsing

## Summary

Parses per-base median sequence quality values from FastQC/MultiQC output on raw paired-end FASTQ reads and applies Tourmaline's fastqc_per_base_sequence_quality_dropoff.py logic to suggest 3' truncation lengths for DADA2/Deblur denoising.

## When to use

Use when raw paired-end Illumina FASTQ reads need DADA2/Deblur truncation lengths derived from per-base quality profiles rather than manual inspection or fixed defaults.

## When NOT to use

- DADA2/Deblur truncation lengths are already fixed in the config and no quality-based re-estimation is desired
- Input is already a denoised feature table or representative sequences rather than raw FASTQ with per-base quality
- Data are single-end or non-Illumina reads whose quality profiles do not map to the Read1/Read2 dropoff logic

## Inputs

- Raw paired-end Illumina FASTQ reads (e.g., 2018 Lake Erie surface water 16S rRNA amplicon data)
- FastQC or MultiQC per-base sequence quality summaries (median quality per position for Read 1 and Read 2)
- Tourmaline config file with dada2_trunc_len_f / dada2pe_trunc_len_r reference parameters

## Outputs

- Suggested forward truncation length in bp (e.g., 240 bp for the Lake Erie data)
- Suggested reverse truncation length in bp (e.g., 190 bp for the Lake Erie data)
- trimming_parameters.tsv with suggested and reference truncation lengths

## How to apply

Parse the per-base median sequence quality values from the quality-control summaries of raw FASTQ reads (e.g., the 2018 Lake Erie 16S paired-end data). For each read direction, scan positions until the median quality falls below a fraction (default 0.90) of its maximum median quality and record that position as the dropoff point, mirroring Tourmaline's fastqc_per_base_sequence_quality_dropoff.py. Run the helper on Read 1 and Read 2 separately to obtain forward and reverse truncation suggestions. Compare the suggested lengths against the dada2_trunc_len_f and dada2_trunc_len_r (dada2pe_trunc_len_r) parameters in the Tourmaline repseqs config, writing results to trimming_parameters.tsv. The goal is a balance between retaining sequence length and removing low-quality 3' tails before denoising.

## Related tools

- **Tourmaline** (Source of the fastqc_per_base_sequence_quality_dropoff.py helper and the config truncation parameters (dada2_trunc_len_f, dada2pe_trunc_len_r)) — github.com/aomlomics/tourmaline
- **QIIME 2** (Hosts the DADA2/Deblur denoising that consumes the suggested truncation lengths)
- **DADA2** (Denoising algorithm whose trunc_len_f/trunc_len_r parameters are informed by the parsed quality dropoff) — github.com/benjjneb/dada2
- **Snakemake** (Workflow system in which the qaqc step and trimming-parameter determination are executed)
- **Python** (Language used to reimplement the dropoff estimator that scans per-base median quality)

## Examples

```
python fastqc_per_base_sequence_quality_dropoff.py --fraction 0.90 --read1 R1.fastq.gz --read2 R2.fastq.gz
```

## Evaluation signals

- Suggested forward truncation length equals 240 bp on the Lake Erie 16S dataset
- Suggested reverse truncation length equals 190 bp on the Lake Erie 16S dataset
- Suggested lengths match the documented DADA2 truncation lengths (dada2_trunc_len_f, dada2pe_trunc_len_r) from the configuration example
- trimming_parameters.tsv is populated with both suggested and reference truncation lengths
- Dropoff position is found where median quality falls below 0.90 of the maximum median quality for each read direction

## Limitations

- The estimator only reports the position where median quality drops below the 0.90 fraction of maximum; it does not account for other truncation criteria such as read overlap requirements for merging
- Quality profiles are dataset-specific, so suggested lengths must be recomputed per run rather than reused across projects
- The helper depends on FastQC/MultiQC output being available for the raw reads; it cannot run without a quality-control summary

## Evidence

- [other] Running fastqc_per_base_sequence_quality_dropoff.py on the 2018 Lake Erie data suggested a forward truncation length of 240 bp and reverse truncation length of 190 bp: [evidence span withheld — anchor status: not_found]
- [other] the script works by determining the position where median per-base sequence quality drops below a fraction (default 0.90) of its maximum value: [evidence span withheld — anchor status: not_found]
- [abstract] Features include multiple-core support, automatic trimming parameter determination from quality scores: "Features include support for multiple cores, automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID)"
- [intro] different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory"
