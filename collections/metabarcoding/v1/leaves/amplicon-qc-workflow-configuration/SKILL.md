---
name: amplicon-qc-workflow-configuration
description: 'Use when configuring Tourmaline/QIIME 2 amplicon QC: paired-end Illumina
  16S FASTQ needs DADA2/Deblur 3'' truncation lengths derived from FastQC/MultiQC
  per-base median quality (dropoff below 0.90 of maximum) instead of manual guesses.'
when_to_use_negative:
- Input is already denoised output (ASV/feature table or representative sequences)
  — truncation lengths only apply to raw/trimmed FASTQ entering the DADA2/Deblur step.
- Running Deblur, which Tourmaline uses in single-end mode — Read 1/Read 2 paired
  truncation lengths from this helper target DADA2 paired-end denoising.
- Per-sample adaptive trimming is needed — the helper suggests one length per direction
  from the full read set's median quality curve, not sample-specific values.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: Tourmaline
  role: Provides the fastqc_per_base_sequence_quality_dropoff.py helper and the repseqs
    config parameters (dada2_trunc_len_f, dada2pe_trunc_len_r) that carry suggested
    truncation lengths into the Snakemake-managed qaqc/repseqs steps
  repo: github.com/aomlomics/tourmaline
- name: FastQC
  role: Produces the per-base sequence quality summaries whose median values the dropoff
    estimator scans for Read 1 and Read 2
- name: MultiQC
  role: Aggregates the FastQC per-base quality output consumed by the truncation-length
    estimator
- name: QIIME 2
  role: Runs the DADA2/Deblur denoising step that consumes the configured truncation
    lengths
  repo: https://qiime2.org
- name: DADA2
  role: Paired-end denoiser whose truncation parameters are set from the suggested
    forward/reverse lengths
  repo: github.com/benjjneb/dada2
- name: Cutadapt
  role: 'Optional primer trimming in the qaqc step preceding truncation-length estimation
    (discard_untrimmed: false, minimum_length: 50)'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:e23522c32477f058eafff44eecc5dbdc2052dd1eb14488174a39ed874fe24801
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cutadapt
  - DADA2
  - FastQC
  - MultiQC
  - Python
  - QIIME 2
  - Tourmaline
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# amplicon-qc-workflow-configuration

## Summary

Configures the Tourmaline (QIIME 2 + Snakemake) amplicon quality-control step: a helper script (fastqc_per_base_sequence_quality_dropoff.py) scans per-base median sequence quality in FastQC/MultiQC output and sets DADA2/Deblur 3' truncation lengths where median quality drops below a default 0.90 fraction of its maximum (240 bp forward, 190 bp reverse for the 2018 Lake Erie 16S data). This makes denoising truncation parameters reproducible and quality-driven rather than manual.

## When to use

Use when configuring Tourmaline/QIIME 2 amplicon QC: paired-end Illumina 16S FASTQ needs DADA2/Deblur 3' truncation lengths derived from FastQC/MultiQC per-base median quality (dropoff below 0.90 of maximum) instead of manual guesses.

## When NOT to use

- Input is already denoised output (ASV/feature table or representative sequences) — truncation lengths only apply to raw/trimmed FASTQ entering the DADA2/Deblur step.
- Running Deblur, which Tourmaline uses in single-end mode — Read 1/Read 2 paired truncation lengths from this helper target DADA2 paired-end denoising.
- Per-sample adaptive trimming is needed — the helper suggests one length per direction from the full read set's median quality curve, not sample-specific values.

## Inputs

- Raw paired-end Illumina FASTQ reads (e.g., 2018 Lake Erie surface-water 16S rRNA amplicon data)
- FastQC/MultiQC per-base sequence quality summaries (median quality per position, Read 1 and Read 2)
- Tourmaline configuration file (repseqs config with DADA2 truncation parameters dada2_trunc_len_f / dada2pe_trunc_len_r)

## Outputs

- Suggested 3' truncation lengths per read direction (e.g., 240 bp forward, 190 bp reverse)
- trimming_parameters.tsv recording the suggested and reference/configured DADA2 truncation lengths
- Updated Tourmaline config parameters feeding the DADA2/Deblur denoising (repseqs) step

## How to apply

Run the Tourmaline qaqc step (optionally trimming primers with Cutadapt, e.g., discard_untrimmed: false, minimum_length: 50) to obtain FastQC/MultiQC per-base sequence quality summaries of the raw paired-end FASTQ reads, as done for the 2018 Lake Erie surface-water 16S rRNA dataset. Reimplement or invoke fastqc_per_base_sequence_quality_dropoff.py: parse the median per-base sequence quality for Read 1 and Read 2, then scan positions until the median quality falls below a fraction (default 0.90) of its maximum value, recording that position as the suggested 3' truncation length. For the Lake Erie data this yields 240 bp (forward) and 190 bp (reverse), described as a balance between sequence length and quality. Transfer the values into the DADA2 truncation parameters in the Tourmaline repseqs config (dada2_trunc_len_f, dada2pe_trunc_len_r) that drive QIIME 2's DADA2 (or Deblur) denoising, and write suggested plus reference lengths to trimming_parameters.tsv. Validate by checking the suggested lengths match the documented DADA2 truncation lengths from the configuration example and that re-running the scan on the same quality summaries is deterministic.

## Related tools

- **Tourmaline** (Provides the fastqc_per_base_sequence_quality_dropoff.py helper and the repseqs config parameters (dada2_trunc_len_f, dada2pe_trunc_len_r) that carry suggested truncation lengths into the Snakemake-managed qaqc/repseqs steps) — github.com/aomlomics/tourmaline
- **FastQC** (Produces the per-base sequence quality summaries whose median values the dropoff estimator scans for Read 1 and Read 2)
- **MultiQC** (Aggregates the FastQC per-base quality output consumed by the truncation-length estimator)
- **QIIME 2** (Runs the DADA2/Deblur denoising step that consumes the configured truncation lengths) — https://qiime2.org
- **DADA2** (Paired-end denoiser whose truncation parameters are set from the suggested forward/reverse lengths) — github.com/benjjneb/dada2
- **Cutadapt** (Optional primer trimming in the qaqc step preceding truncation-length estimation (discard_untrimmed: false, minimum_length: 50))

## Examples

```
./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N
```

## Evaluation signals

- Suggested lengths match the documented DADA2 truncation lengths (forward and reverse) from the Tourmaline configuration example (240 bp forward, 190 bp reverse for the Lake Erie 16S run).
- Each reported dropoff position equals the first per-base position where median quality < 0.90 x maximum median quality in the FastQC/MultiQC summary for that read direction.
- trimming_parameters.tsv contains both suggested and reference/configured truncation lengths for forward and reverse reads.
- dada2_trunc_len_f and dada2pe_trunc_len_r in the Tourmaline repseqs config equal the suggested values before the repseqs (DADA2) step runs.
- Re-running the helper on the same FastQC/MultiQC quality summaries reproduces identical lengths (deterministic position scan).

## Limitations

- The default 0.90 dropoff fraction is a heuristic balancing sequence length against quality; suggested lengths are dataset-specific (240/190 bp for the Lake Erie 16S data) and must be re-derived for each new dataset.
- The helper returns a single truncation length per direction derived from median per-base quality of the full read set, not sample-specific or per-read values.
- Deblur is used in single-end mode in Tourmaline, so Read 1/Read 2 paired truncation parameters from this helper are specific to DADA2 paired-end runs.

## Evidence

- [abstract] automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID): "automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID)"
- [other] Running fastqc_per_base_sequence_quality_dropoff.py on the 2018 Lake Erie data suggested a forward truncation length of 240 bp and reverse truncation length of 190 bp: [evidence span withheld — anchor status: not_found]
- [other] the position where median per-base sequence quality drops below a fraction (default 0.90) of its maximum value in FastQC/MultiQC output for Read 1 and Read 2: [evidence span withheld — anchor status: not_found]
- [other] Compare the suggested lengths against the DADA2 truncation parameters in the Tourmaline repseqs config (dada2_trunc_len_f, dada2pe_trunc_len_r): [evidence span withheld — anchor status: not_found]
- [other] the suggested truncation lengths match the documented DADA2 truncation lengths (forward and reverse) from the configuration example: [evidence span withheld — anchor status: not_found]
- [other] Write the suggested and reference truncation lengths to trimming_parameters.tsv: [evidence span withheld — anchor status: not_found]
- [readme] Tourmaline 2 is an amplicon sequence processing workflow for Illumina sequence data that uses [QIIME 2](https://qiime2.org) and the software packages it wraps.: "Tourmaline 2 is an amplicon sequence processing workflow for Illumina sequence data that uses [QIIME 2](https://qiime2.org) and the software packages it wraps."
- [readme] ./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [methods] Optionally trims primer sequences from raw reads.: "Optionally trims primer sequences from raw reads."
