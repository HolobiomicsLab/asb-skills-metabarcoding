---
name: amplicon-read-preprocessing
description: Use when raw paired-end amplicon fastq files (16S, ITS, etc.) need to
  be turned into denoised, chimera-checked ASV tables with taxonomy, on a laptop or
  a cluster with limited cores.
when_to_use_negative:
- Input is already a feature/ASV table or taxonomy-assigned OTU table — dadasnake
  starts from raw fastq files.
- Data are not amplicon reads (e.g. shotgun metagenomics or genomes) — the pipeline
  is specific to amplicon sequencing data.
- A non-Snakemake-friendly single-node environment without conda and no need for ASV
  inference — installation assumes conda and snakemake >= 5.9.1.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: dadasnake
  role: single-command Snakemake workflow orchestrating the whole preprocessing from
    raw fastq to taxonomically assigned OTU tables
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: workflow engine that dispatches parallel jobs across cores/cluster nodes (thread
    cap set via -t)
- name: DADA2
  role: quality filtering and trimming, error estimation and denoising to exact sequence
    variants, chimera removal
- name: cutadapt
  role: primer removal and removal of poly-G-tails
- name: seqtk
  role: optional downsampling of reads per sample
  repo: https://github.com/lh3/seqtk
- name: ITSx
  role: optional filtering of ITS reads to a specified region (e.g. ITS1)
- name: mothur
  role: Bayesian taxonomic classification against formatted databases (e.g. UNITE
    for fungal ITS)
- name: conda
  role: environment management; dadasnake runs in a single conda environment with
    per-step environments
merged_aliases:
- amplicon-sequencing-data-processing
- amplicon-sequencing-qc-preprocessing
merged_alias_records:
- alias: amplicon-sequencing-data-processing
  slug: amplicon-sequencing-data-processing
  cosine_score: 0.8294
  method: embedding-cosine
  decision: needs_review
- alias: amplicon-sequencing-qc-preprocessing
  slug: amplicon-sequencing-qc-preprocessing
  cosine_score: 0.8522
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:4402d439e47d032001bd4c31703f93208f23704251b01aff0e4d37c537cf5fba
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
  - seqtk
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# amplicon-read-preprocessing

## Summary

Preprocess raw amplicon FASTQ reads (primer/poly-G removal, quality filtering and trimming, error estimation, ASV inference, read-pair merging, chimera removal) with dadasnake, a single-command Snakemake pipeline wrapping DADA2, configurable via a YAML config and a tab-separated sample table.

## When to use

Use when raw paired-end amplicon fastq files (16S, ITS, etc.) need to be turned into denoised, chimera-checked ASV tables with taxonomy, on a laptop or a cluster with limited cores.

## When NOT to use

- Input is already a feature/ASV table or taxonomy-assigned OTU table — dadasnake starts from raw fastq files.
- Data are not amplicon reads (e.g. shotgun metagenomics or genomes) — the pipeline is specific to amplicon sequencing data.
- A non-Snakemake-friendly single-node environment without conda and no need for ASV inference — installation assumes conda and snakemake >= 5.9.1.

## Inputs

- raw paired-end amplicon FASTQ read files (e.g. 16S V4 or ITS1) from a single raw_directory
- tab-separated sample table with 'library', 'r1_file', 'r2_file' columns (plus run IDs for run-specific error models)
- YAML config file (output directory, raw_directory, sample_table path, paired, trunc_length, min_overlap, taxonomy database settings)
- reference taxonomy databases (e.g. SILVA for 16S, UNITE/fungalTraits for ITS)

## Outputs

- final ASV table in biom format
- R/phyloseq objects
- FASTA file of ASV sequences and tab-separated ASV tables
- taxonomically assigned OTU/ASV tables
- figures and tables reporting intermediary steps and statistics, plus a 'workflow.done' completion marker

## How to apply

Obtain raw fastq files and build a tab-separated sample table with 'library', 'r1_file' and 'r2_file' columns, adding run IDs so run-specific DADA2 error models can be estimated. Write a human-readable YAML config setting raw_directory, output directory, paired=true, trunc_length (e.g. 90 bp for the EMP subset), primer removal via cutadapt (including poly-G-tail removal), DADA2 quality filtering/trimming, error estimation and denoising, pair merging (min_overlap at least 20 bp for ITS1), chimera removal, optional ITSx filtering and mothur or DADA2 taxonomic classification. Validate everything with a dry-run (dadasnake -d -r config.yaml) before committing compute, then launch with the dadasnake wrapper, capping threads with -t (1, 4, 15, or up to 50 cores). Run completion is signaled by a 'workflow.done' marker file; final ASV tables are handed off as biom format, phyloseq objects, fasta, and tab-separated tables. Snakemake parallelization gives substantial speedups: the 24-sample 16S dataset dropped from ~150 min on 1 core to ~100 min on 4 cores, and the 267-sample ITS1 dataset from just under 4 hours on 4 cores to roughly half that on 15 cores.

## Related tools

- **dadasnake** (single-command Snakemake workflow orchestrating the whole preprocessing from raw fastq to taxonomically assigned OTU tables) — https://github.com/a-h-b/dadasnake
- **Snakemake** (workflow engine that dispatches parallel jobs across cores/cluster nodes (thread cap set via -t))
- **DADA2** (quality filtering and trimming, error estimation and denoising to exact sequence variants, chimera removal)
- **cutadapt** (primer removal and removal of poly-G-tails)
- **seqtk** (optional downsampling of reads per sample) — https://github.com/lh3/seqtk
- **ITSx** (optional filtering of ITS reads to a specified region (e.g. ITS1))
- **mothur** (Bayesian taxonomic classification against formatted databases (e.g. UNITE for fungal ITS))
- **conda** (environment management; dadasnake runs in a single conda environment with per-step environments)

## Examples

```
dadasnake -l -n RUN1 -r config.yaml -t 4
```

## Evaluation signals

- A 'workflow.done' marker file appears in the output directory at completion.
- Dry-run (dadasnake -d) validates input paths, sample table formatting, and step compatibility without errors.
- Recorded wall-clock runtimes are consistent with the reported benchmarks (~150 min on 1 core / ~100 min on 4 cores for 2.9M reads; just under 4 hours on 4 cores / ~2x speedup at 15 cores for 46.8M reads; ~87 hours at <=50 cores for 2.1B reads).
- Final ASV table exists in all hand-off formats (biom, phyloseq, fasta, tab-separated).
- Intermediary statistics figures/tables (per-step read retention) are generated and plausible across the pipeline steps.

## Limitations

- Very large datasets need bigmem nodes: the EMP run (2.1B reads, >27,000 samples) required >190 GB peak memory for the >700,000-ASV table.
- Snakemake versions prior to 5.9.1 are unsupported; cluster submission requires scheduler-specific config (slurm or uge) and correct VARIABLE_CONFIG settings.
- Workflow described in the paper is a frozen Zenodo version; taxonomy databases (SILVA, UNITE, fungalTraits v1.2) must be downloaded and configured separately.

## Evidence

- [abstract] single-command pipeline: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants"
- [methods] primer removal: "primer removal and removal of poly-G-tails - using cutadapt"
- [methods] DADA2 steps: "error estimation & denoising - using DADA2, including Novaseq-enabled models"
- [methods] downsampling option: "optional downsampling of reads per sample - using seqtk"
- [readme] wrapper setup: "MAX_THREADS - set this to the maximum number of cores you want to be using in a run. If you don't set this, the default will be 50."
- [readme] snakemake version: "Dadasnake will run with Snakemake version >= 5.9.1 and hasn't been tested with any previous versions."
