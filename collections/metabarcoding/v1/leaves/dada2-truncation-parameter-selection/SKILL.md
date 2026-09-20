---
name: dada2-truncation-parameter-selection
description: Use when preparing raw paired-end FASTQ amplicon reads for DADA2 denoising
  and you need data-driven forward/reverse truncation lengths instead of manually
  inspecting quality plots.
when_to_use_negative:
- Input is already a denoised feature table or ASV sequence artifact — truncation
  must be decided before DADA2/Deblur runs
- The reads are not Illumina amplicon FASTQ data with available per-base quality summaries
  from FastQC/MultiQC
- A truncation length is already fixed by the study design (e.g., amplicon-size constraints
  documented in the assay definition), so the quality-based suggestion is not needed
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Tourmaline
  role: Provides the fastqc_per_base_sequence_quality_dropoff.py helper script that
    implements automatic determination of trimming parameters using quality scores
  repo: https://github.com/aomlomics/tourmaline
- name: DADA2
  role: Denoising algorithm that consumes the suggested truncation lengths (dada2_trunc_len_f,
    dada2_trunc_len_r) during the qaqc/repseqs steps
  repo: https://github.com/benjjneb/dada2
- name: Python
  role: Language in which the truncation-length estimator helper is reimplemented
    and run
- name: QIIME 2
  role: Runs the DADA2 or Deblur denoising algorithm with the selected truncation
    parameters
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:213a7b3144e496aa4ffb74aa0e4c627059b1b3d1bf894695ce77391032574431
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Cutadapt
  - DADA2
  - Python
  - QIIME 2
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dada2-truncation-parameter-selection

## Summary

Determines DADA2/Deblur 3' truncation lengths by parsing per-base median sequence quality from FastQC/MultiQC output and finding where median quality falls below a fraction (default 0.90) of its maximum, as implemented in Tourmaline's fastqc_per_base_sequence_quality_dropoff.py helper script.

## When to use

Use when preparing raw paired-end FASTQ amplicon reads for DADA2 denoising and you need data-driven forward/reverse truncation lengths instead of manually inspecting quality plots.

## When NOT to use

- Input is already a denoised feature table or ASV sequence artifact — truncation must be decided before DADA2/Deblur runs
- The reads are not Illumina amplicon FASTQ data with available per-base quality summaries from FastQC/MultiQC
- A truncation length is already fixed by the study design (e.g., amplicon-size constraints documented in the assay definition), so the quality-based suggestion is not needed

## Inputs

- Raw demultiplexed paired-end Illumina FASTQ reads (e.g., Lake Erie 16S amplicon data)
- FastQC or MultiQC per-base sequence quality summaries (median quality per base position for Read 1 and Read 2)

## Outputs

- Suggested forward truncation length for DADA2 (e.g., 240 bp for the Lake Erie 16S data)
- Suggested reverse truncation length for DADA2 (e.g., 190 bp for the Lake Erie 16S data)
- trimming_parameters.tsv recording suggested and reference (config) truncation lengths

## How to apply

Parse the per-base median sequence quality values from the quality-control summaries of raw paired-end FASTQ reads (e.g., FastQC/MultiQC output for Read 1 and Read 2). For each read direction, scan per-base positions until the median quality drops below a fraction (default 0.90) of its maximum median quality, and record that position as the suggested truncation length — this balances sequence length retained against quality at the read 3' end. On the 2018 Lake Erie 16S surface-water dataset this procedure suggested a forward truncation of 240 bp and a reverse truncation of 190 bp. Compare the suggested lengths against the DADA2 truncation parameters in the Tourmaline repseqs config (dada2_trunc_len_f, dada2_trunc_len_r), then write the suggested and reference lengths to trimming_parameters.tsv and use the values to set truncation parameters before denoising.

## Related tools

- **Tourmaline** (Provides the fastqc_per_base_sequence_quality_dropoff.py helper script that implements automatic determination of trimming parameters using quality scores) — https://github.com/aomlomics/tourmaline
- **DADA2** (Denoising algorithm that consumes the suggested truncation lengths (dada2_trunc_len_f, dada2_trunc_len_r) during the qaqc/repseqs steps) — https://github.com/benjjneb/dada2
- **Python** (Language in which the truncation-length estimator helper is reimplemented and run)
- **QIIME 2** (Runs the DADA2 or Deblur denoising algorithm with the selected truncation parameters)

## Examples

```
python fastqc_per_base_sequence_quality_dropoff.py --input multiqc_fastqc/ --fraction 0.90
```

## Evaluation signals

- The suggested forward truncation length (240 bp) and reverse truncation length (190 bp) for the 2018 Lake Erie 16S data match the documented DADA2 truncation lengths in the Tourmaline repseqs config example (dada2_trunc_len_f, dada2_trunc_len_r)
- The recorded position for each read direction is the first per-base position where median quality falls below 0.90 of the maximum median quality
- Read 1 and Read 2 are evaluated independently, producing separate forward and reverse truncation values
- Output trimming_parameters.tsv contains both the suggested lengths and the reference config lengths in a comparable format
- Running the helper on a different dataset yields positions that monotonically correspond to the quality-dropoff profiles visible in the FastQC/MultiQC quality summaries

## Limitations

- The 0.90 fraction is a default balancing sequence length against quality; datasets with different quality profiles may need a different fraction
- Truncation lengths suggested from median quality may not account for reverse-read overlap requirements when merging paired reads in DADA2
- The method depends on the availability and format of FastQC/MultiQC per-base quality summaries; it cannot run directly on raw FASTQ without first computing those summaries

## Evidence

- [abstract] automatic determination of trimming parameters using quality scores: "Features include support for multiple cores, automatic determination of trimming parameters using quality scores"
- [other] determining the position where median per-base sequence quality drops below a fraction (default 0.90) of its maximum value in FastQC/MultiQC output for Read 1 and Read 2: [evidence span withheld — anchor status: not_found]
- [other] forward truncation length of 240 bp and reverse truncation length of 190 bp: [evidence span withheld — anchor status: not_found]
- [readme] tourmaline.sh qaqc invocation: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
