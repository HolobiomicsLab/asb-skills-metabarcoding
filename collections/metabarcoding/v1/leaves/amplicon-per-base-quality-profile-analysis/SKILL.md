---
name: amplicon-per-base-quality-profile-analysis
description: Use when raw paired-end FASTQ reads (e.g. Illumina 16S amplicon data)
  need DADA2 truncation parameters, or when Tourmaline's qaqc trimming-parameter automation
  must be reproduced or verified from quality profiles.
when_to_use_negative:
- Input is already a denoised feature table or representative sequences — truncation
  is only relevant before DADA2/Deblur denoising in the Tourmaline qaqc step
- Reads are not paired-end Illumina amplicon data with per-base quality summaries
  available (no FastQC/MultiQC quality profile to scan)
- An external tool has already generated trimming parameters and you only need to
  feed pre-processed data into a Tourmaline step, which Tourmaline 2 supports directly
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: Tourmaline
  role: Provides the fastqc_per_base_sequence_quality_dropoff.py helper and the qaqc
    step that performs automatic determination of trimming parameters using quality
    scores
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the DADA2 or Deblur denoising that consumes the suggested truncation
    lengths
- name: DADA2
  role: Denoising algorithm whose truncation-length parameters are determined by this
    analysis
  repo: github.com/benjjneb/dada2
- name: Snakemake
  role: Workflow management system orchestrating the Tourmaline qaqc step that wraps
    this quality-based trimming-parameter determination
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:d843e2b50db8b2bc6a5fd08e5183bd2d6193dc17898222c51636d41f8002a4e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Cutadapt
  - DADA2
  - Python
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# amplicon-per-base-quality-profile-analysis

## Summary

Analyzes per-base sequence quality profiles from FastQC/MultiQC summaries of paired-end Illumina amplicon reads to suggest 3' truncation lengths for DADA2/Deblur denoising, replicating Tourmaline's fastqc_per_base_sequence_quality_dropoff.py logic.

## When to use

Use when raw paired-end FASTQ reads (e.g. Illumina 16S amplicon data) need DADA2 truncation parameters, or when Tourmaline's qaqc trimming-parameter automation must be reproduced or verified from quality profiles.

## When NOT to use

- Input is already a denoised feature table or representative sequences — truncation is only relevant before DADA2/Deblur denoising in the Tourmaline qaqc step
- Reads are not paired-end Illumina amplicon data with per-base quality summaries available (no FastQC/MultiQC quality profile to scan)
- An external tool has already generated trimming parameters and you only need to feed pre-processed data into a Tourmaline step, which Tourmaline 2 supports directly

## Inputs

- Raw paired-end Illumina FASTQ reads (e.g. 2018 Lake Erie surface-water 16S rRNA amplicon data)
- FastQC/MultiQC per-base sequence quality summaries (quality-control module output)
- Tourmaline configuration file with DADA2 truncation parameters (dada2_trunc_len_f, dada2pe_trunc_len_r)

## Outputs

- Suggested 3' truncation lengths for Read 1 (forward) and Read 2 (reverse), e.g. 240 bp and 190 bp for the Lake Erie 16S dataset
- trimming_parameters.tsv containing suggested and reference (config) DADA2 truncation lengths

## How to apply

Parse the per-base median sequence quality values from the quality-control summaries of the raw Lake Erie 16S paired-end FASTQ reads. For each read direction (Read 1 and Read 2), scan per-base positions until the median quality falls below a fraction (default 0.90) of its maximum median quality, and record that position as the suggested truncation length — this balances sequence length retained against quality dropoff before DADA2 denoising. For the 2018 Lake Erie 16S data this yields a forward truncation length of 240 bp and a reverse truncation length of 190 bp. Run the helper on the full read set, then compare the suggested lengths against the DADA2 truncation parameters in the Tourmaline repseqs config (dada2_trunc_len_f, dada2pe_trunc_len_r) and write both the suggested and reference values to trimming_parameters.tsv for parameter comparison across runs.

## Related tools

- **Tourmaline** (Provides the fastqc_per_base_sequence_quality_dropoff.py helper and the qaqc step that performs automatic determination of trimming parameters using quality scores) — github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the DADA2 or Deblur denoising that consumes the suggested truncation lengths)
- **DADA2** (Denoising algorithm whose truncation-length parameters are determined by this analysis) — github.com/benjjneb/dada2
- **Snakemake** (Workflow management system orchestrating the Tourmaline qaqc step that wraps this quality-based trimming-parameter determination)

## Examples

```
./tourmaline.sh --step qaqc --configfile config/config.yml --cores 8
```

## Evaluation signals

- The suggested forward truncation length for the 2018 Lake Erie 16S data equals 240 bp and the reverse equals 190 bp, matching the documented DADA2 truncation lengths
- The scan threshold is respected: each suggested length is the position where median per-base quality first drops below 0.90 of the maximum median quality for that read direction
- trimming_parameters.tsv contains both suggested and config reference values (dada2_trunc_len_f, dada2pe_trunc_len_r) in a consistent schema
- Suggested lengths are monotonically non-decreasing scan positions and do not exceed the read lengths of the input FASTQ files

## Limitations

- The default quality-dropoff fraction (0.90) is a heuristic; different fractions yield different truncation lengths and must be tuned per dataset
- The method depends on FastQC/MultiQC median quality summaries, so it cannot be applied without per-base quality-control summaries of the raw reads
- No changelog was found for the Tourmaline repository, so version-specific behavior changes in the helper script are not documented

## Evidence

- [methods] Automatic determination of trimming parameters using quality scores: [evidence span withheld — anchor status: not_found]
- [abstract] automatic determination of trimming parameters from quality scores: [evidence span withheld — anchor status: not_found]
- [readme] Tourmaline 2 is an amplicon sequence processing workflow for Illumina sequence data that uses [QIIME 2]: [evidence span withheld — anchor status: not_found]
- [other] median per-base sequence quality drops below a fraction (default 0.90) of its maximum value in FastQC/MultiQC output for Read 1 and Read 2: [evidence span withheld — anchor status: not_found]
