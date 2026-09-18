---
name: amplicon-sequence-denoising-dada2
description: Use when raw demultiplexed paired-end Illumina FASTQ reads need error
  correction and chimera removal into a feature table/representative-sequence pair,
  with trimming parameters determined automatically from quality scores.
when_to_use_negative:
- Data are single-end reads where the Deblur denoising algorithm is the appropriate
  Tourmaline option
- Input is already a denoised feature table or representative sequence artifact (start
  Tourmaline at the repseqs or taxonomy step instead)
- Processing amplicons from platforms other than Illumina sequence data
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: Tourmaline
  role: Snakemake-based workflow that runs DADA2 through the dada2_pe_denoise pseudo-rule
    and the tourmaline.sh script
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Provides the DADA2 denoising plugin invoked by the workflow
- name: DADA2
  role: Denoising algorithm that converts FASTQ reads into ASV feature table and representative
    sequences
  repo: https://github.com/benjjneb/dada2
- name: Snakemake
  role: Workflow management system that manages commands, inputs, and outputs of the
    denoising step
- name: Docker
  role: Containerized environment in which the workflow (including DADA2) runs on
    Linux/macOS
merged_aliases:
- amplicon-sequence-denoising-with-dada2
merged_alias_records:
- alias: amplicon-sequence-denoising-benchmarking
  slug: amplicon-sequence-denoising-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:a290886b4da9eefbb553ad9b241ed125fc930b9787469cb09454639c37bd2274
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - DADA2
  - DEBLUR
  - Docker
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# amplicon-sequence-denoising-dada2

## Summary

Denoises Illumina amplicon reads into amplicon sequence variants (ASVs) using the DADA2 algorithm as implemented in QIIME 2, executed through the Tourmaline Snakemake workflow (rule dada2_pe_denoise for paired-end data). It produces the feature table and representative sequence artifacts that anchor all downstream taxonomy, diversity, and reporting steps.

## When to use

Use when raw demultiplexed paired-end Illumina FASTQ reads need error correction and chimera removal into a feature table/representative-sequence pair, with trimming parameters determined automatically from quality scores.

## When NOT to use

- Data are single-end reads where the Deblur denoising algorithm is the appropriate Tourmaline option
- Input is already a denoised feature table or representative sequence artifact (start Tourmaline at the repseqs or taxonomy step instead)
- Processing amplicons from platforms other than Illumina sequence data

## Inputs

- Raw demultiplexed paired-end Illumina FASTQ files (e.g., PRJNA679730 Lake Erie 16S rRNA data, ~96 samples, ~11.5 million paired reads)
- Sample manifest file
- Sample metadata file
- qaqc step config file (includes DADA2 truncation/trimming parameters)

## Outputs

- QIIME 2 feature table artifact
- QIIME 2 representative sequence artifact
- Downstream inputs for taxonomy assignment, diversity analyses, and the HTML report

## How to apply

Place the raw FASTQ files, sample manifest, sample metadata, and reference database in the working directory and configure the three step config files (qaqc, repseqs, taxonomy); the qaqc config controls DADA2 truncation lengths, with automatic determination of trimming parameters from quality scores as the default. Activate the Tourmaline conda environments (Snakemake environment plus the bundled QIIME 2 environment) or use the Docker container, then run the Snakemake pseudo-rule dada2_pe_denoise (or ./tourmaline.sh --step qaqc --cores N), which invokes QIIME 2's DADA2 paired-end denoising. DADA2 can be swapped for Deblur (single-end) depending on the read mode; the rule outputs the feature table and representative sequence artifacts that feed taxonomy assignment (dada2_pe_taxonomy), diversity analyses (dada2_pe_diversity), and the HTML report. Multi-core support lets larger runs (e.g., ~11.5M paired reads from PRJNA679730) scale from ~5 h on 1 core to ~2 h on 8 cores.

## Related tools

- **Tourmaline** (Snakemake-based workflow that runs DADA2 through the dada2_pe_denoise pseudo-rule and the tourmaline.sh script) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Provides the DADA2 denoising plugin invoked by the workflow)
- **DADA2** (Denoising algorithm that converts FASTQ reads into ASV feature table and representative sequences) — https://github.com/benjjneb/dada2
- **Snakemake** (Workflow management system that manages commands, inputs, and outputs of the denoising step)
- **Docker** (Containerized environment in which the workflow (including DADA2) runs on Linux/macOS)

## Examples

```
./tourmaline.sh --step qaqc --configfile config-qaqc --cores 8
```

## Evaluation signals

- The run directory contains the DADA2 feature table and representative sequences artifacts after dada2_pe_denoise completes
- Downstream rules (dada2_pe_taxonomy, dada2_pe_diversity, dada2_pe_report) run without missing-input errors, confirming denoising outputs are valid QIIME 2 artifacts
- Truncation/trimming parameters in the qaqc config match the automatic determination from quality scores
- End-to-end runtime is consistent with multi-core expectations (~5 h with --cores 1 vs ~2 h with --cores 8 for the full Lake Erie dataset)
- Different DADA2 truncation-length runs produce distinct, comparable run directories via the helper script for parameter optimization

## Limitations

- Deblur is single-end only, so paired-end datasets must use DADA2 in this workflow
- DADA2 truncation-length choices strongly affect results; runs with different parameters need separate directory copies for comparison
- Multi-core speedup is partial: 8 cores reduce the full Lake Erie run only from ~5 h to ~2 h

## Evidence

- [other] Tourmaline uses QIIME 2 DADA2 paired-end denoising to produce the feature table and representative sequence artifacts via the dada2_pe_denoise rule.: [evidence span withheld — anchor status: not_found]
- [abstract] Tourmaline runs either DADA2 or Deblur denoising through QIIME 2.: "it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm"
- [abstract] Trimming parameters are determined automatically from quality scores.: "automatic determination of trimming parameters using quality scores"
- [methods] Denoising (DADA2, dada2pe) is timed alone through tourmaline.sh/Snakemake with --cores 1 and --cores 8 for runtime benchmarking.: [evidence span withheld — anchor status: not_found]
- [readme] tourmaline.sh is the main interface for running workflow steps with specified config files and core counts.: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [intro] Runs with different DADA2 truncation lengths can be compared using the helper script that copies the Tourmaline directory.: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script that makes a new copy of the Tourmaline directory"
