---
name: dada2-asv-inference
description: Use when raw paired-end amplicon fastq data (16S, ITS1/ITS2) must be
  converted from reads to a taxonomically assigned ASV table, especially for 24–27,000
  sample datasets on limited cores.
when_to_use_negative:
- Input is already a feature/ASV table or OTU table — dadasnake starts from raw fastq
  files
- Single-genome or shotgun metagenomic reads that require assembly (e.g. SPAdes) rather
  than amplicon denoising
- Data needing denoising methods other than DADA2-style exact sequence variant inference
  (e.g. UPARSE OTU clustering) without reconfiguration
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: dadasnake
  role: 1-command Snakemake wrapper that executes the DADA2-based preprocessing, ASV
    delineation, and taxonomic classification steps
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Dispatches parallel jobs (e.g. -t 4 vs -t 15 cores) and drives the workflow
    execution
- name: DADA2
  role: Performs quality filtering and trimming, error estimation and denoising (with
    Novaseq-enabled models), and chimera removal to infer exact sequence variants
- name: cutadapt
  role: Removes primers and poly-G tails (minimum overlap 10, maximum 2 primers per
    end, 0.2 mismatch rate)
- name: ITSx
  role: Extracts ITS1/ITS2 subregions from ASVs with e-value 1.00E-05 and filters
    for target taxon (e.g. fungi)
- name: mothur
  role: Bayesian taxonomic classification of ASVs (cutoff 60) against SILVA or UNITE-formatted
    databases
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:49bdce04037d4be397e9df70e712b3f9147e397627d610fafbfb89c5b4cf3597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - DADA2
  - ITSx
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# dada2-asv-inference

## Summary

Delineate exact sequence variants (ASVs) from raw amplicon fastq files by running the DADA2-based quality filtering, error estimation/denoising, chimera removal, and taxonomic classification steps as orchestrated by the dadasnake Snakemake pipeline.

## When to use

Use when raw paired-end amplicon fastq data (16S, ITS1/ITS2) must be converted from reads to a taxonomically assigned ASV table, especially for 24–27,000 sample datasets on limited cores.

## When NOT to use

- Input is already a feature/ASV table or OTU table — dadasnake starts from raw fastq files
- Single-genome or shotgun metagenomic reads that require assembly (e.g. SPAdes) rather than amplicon denoising
- Data needing denoising methods other than DADA2-style exact sequence variant inference (e.g. UPARSE OTU clustering) without reconfiguration

## Inputs

- Raw paired-end amplicon fastq files (16S V4, ITS1, or ITS2)
- Tab-separated sample table with 'library', 'r1_file', and 'r2_file' column headers (plus run IDs for run-specific error models)
- YAML configuration file (config.yaml) declaring raw_directory, sample_table, paired flag, ITSx/ITS region, merging min_overlap, and output directory
- Reference taxonomy databases (e.g. SILVA silva_nr99_v138_train_set.fa.gz for 16S, UNITE v8 for ITS)

## Outputs

- ASV table and taxonomy table
- biom-format tables and R/phyloseq objects
- FASTA file of ASV sequences and tab-separated count/taxonomy tables
- 'workflow.done' marker file plus per-rule log files and intermediary statistics figures
- runtime benchmark records (e.g. runtime_benchmark.tsv/csv)

## How to apply

Retrieve the raw fastq files (e.g. from NCBI SRA BioProjects PRJNA517390, PRJNA272747, PRJNA626434) and build a tab-separated sample table with the required 'library', 'r1_file', and 'r2_file' headers, adding run IDs so run-specific DADA2 error models can be estimated. Write a dadasnake config.yaml with paired set to true and the desired steps (cutadapt primer removal, DADA2 quality filtering and trimming, error estimation and denoising including Novaseq-enabled models, chimera removal, ITSx extraction, mothur Bayesian classification against SILMA/UNITE train sets such as silva_nr99_v138_train_set.fa.gz). Validate the configuration and inputs with a dryrun ('dadasnake -d' or 'dadasnake -d -r config.yaml') before committing compute, then launch with a thread cap via -t (e.g. -t 1, -t 4, -t 15, or -t 50 on a cluster), relying on Snakemake parallelization (about 2x speedup from 4 to 15 cores; ~150 min at 1 core vs ~100 min at 4 cores for the 24-sample 2.9M-read dataset). Completion is marked by a 'workflow.done' file in the output directory, and final tables are handed off in biom format, R/phyloseq objects, fasta, and tab-separated tables. For ITS data, enable ITSx (e.g. region ITS1, e-value 1.00E-05) and mothur classification with cutoff 60 against UNITE v8.

## Related tools

- **dadasnake** (1-command Snakemake wrapper that executes the DADA2-based preprocessing, ASV delineation, and taxonomic classification steps) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Dispatches parallel jobs (e.g. -t 4 vs -t 15 cores) and drives the workflow execution)
- **DADA2** (Performs quality filtering and trimming, error estimation and denoising (with Novaseq-enabled models), and chimera removal to infer exact sequence variants)
- **cutadapt** (Removes primers and poly-G tails (minimum overlap 10, maximum 2 primers per end, 0.2 mismatch rate))
- **ITSx** (Extracts ITS1/ITS2 subregions from ASVs with e-value 1.00E-05 and filters for target taxon (e.g. fungi))
- **mothur** (Bayesian taxonomic classification of ASVs (cutoff 60) against SILVA or UNITE-formatted databases)

## Examples

```
dadasnake -l -n RUN1 -r config.yaml -t 4
```

## Evaluation signals

- A 'workflow.done' marker file appears in the output directory and per-rule log files show all steps completed
- Dryrun ('dadasnake -d') passes, confirming sample table headers ('library', 'r1_file', 'r2_file'), input paths, and write permissions
- Benchmark consistency: 24-sample/2.9M-read dataset completes in ~150 min at -t 1 and ~100 min at -t 4; 267-sample/46.8M-read ITS1 dataset in just under 4 h at -t 4 with ~2x speedup at -t 15
- Mock community validation: exactly 19 fungal ASVs recovered with 2 Aspergillus and 3 Fusarium strains (1 Fusarium misclassified) and the known Rhizophagus irregularis diversity overestimation
- Final outputs include all standard hand-off formats (biom, phyloseq, fasta, tab-separated tables)

## Limitations

- Dadasnake requires Snakemake version >= 5.9.1 and has not been tested with previous versions
- Pair merging needs a min_overlap of at least 20 bp; ITSx classification depends on the chosen ITS region and fungal taxon filter (1 detected region minimum)
- Large-scale runs are memory-hungry: the >27,000-sample EMP run needed >190 GB peak RAM and ~87 real hours on <=50 cores, so bigmem node settings (NORMAL_MEM_EACH, BIGMEM_MEM_EACH) matter
- Run-specific DADA2 error models require run IDs in the sample table; without them error estimation is pooled

## Evidence

- [methods] error estimation & denoising - using DADA2, including Novaseq-enabled models: "error estimation & denoising - using DADA2, including Novaseq-enabled models"
- [methods] chimera removal - using DADA2: "chimera removal - using DADA2"
- [methods] primer removal and removal of poly-G-tails - using cutadapt: "primer removal and removal of poly-G-tails - using cutadapt"
- [abstract] dadasnake is a user-friendly, 1-command Snakemake pipeline: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants"
- [readme] Snakemake version requirement: "Dadasnake will run with Snakemake version >= 5.9.1 and hasn't been tested with any previous versions."
- [readme] single-command execution and configuration: "Running dadasnake could not be easier: it is called by a single command from the command line. With a human-readable configuration file and a simple sample table,"
