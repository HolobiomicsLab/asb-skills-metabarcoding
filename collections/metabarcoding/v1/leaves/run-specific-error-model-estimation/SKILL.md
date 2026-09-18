---
name: run-specific-error-model-estimation
description: Use when raw amplicon FASTQ files originate from multiple sequencing
  runs or platforms (e.g. Illumina runs incl. NovaSeq) and DADA2 denoising should
  learn per-run error profiles rather than a single pooled model.
when_to_use_negative:
- Input is already a feature/ASV table or processed reads — error models are estimated
  from raw FASTQ only
- All samples come from a single sequencing run with homogeneous quality profiles,
  where per-run splitting adds no benefit
- The data are not amplicon sequencing reads (e.g. shotgun metagenomes), which dadasnake's
  DADA2-based workflow does not target
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: dadasnake
  role: 1-command Snakemake pipeline that organizes the sample table run IDs and dispatches
    the DADA2 error estimation and denoising steps
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: performs quality filtering and trimming, error estimation & denoising (including
    NovaSeq-enabled models) and chimera removal
  repo: https://github.com/a-h-b/dadasnake
- name: cutadapt
  role: upstream primer removal and removal of poly-G-tails before error model estimation
- name: Snakemake
  role: orchestrates the error-estimation and denoising jobs across cluster cores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:6398735cd3e9f363cf48b6f0c1625fd796d64558e68f0fc5cba4ee89fd44c1d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 12
---

# run-specific-error-model-estimation

## Summary

Estimates DADA2 error models separately for each sequencing run in an amplicon dataset by encoding run IDs in the dadasnake sample table, so that quality filtering, denoising and ASV delineation use error parameters learned from the reads of the same instrument run (including NovaSeq-enabled models).

## When to use

Use when raw amplicon FASTQ files originate from multiple sequencing runs or platforms (e.g. Illumina runs incl. NovaSeq) and DADA2 denoising should learn per-run error profiles rather than a single pooled model.

## When NOT to use

- Input is already a feature/ASV table or processed reads — error models are estimated from raw FASTQ only
- All samples come from a single sequencing run with homogeneous quality profiles, where per-run splitting adds no benefit
- The data are not amplicon sequencing reads (e.g. shotgun metagenomes), which dadasnake's DADA2-based workflow does not target

## Inputs

- raw amplicon paired-end FASTQ files (r1/r2) placed in a single raw_directory
- tab-separated sample table with columns library, r1_file, r2_file and run IDs
- yaml configuration file (trunc_length, paired processing, output directory)
- optional sequencing-platform/reference resources such as SILVA for 16S or UNITE for ITS taxonomy

## Outputs

- per-run DADA2 error model fits used for denoising
- delineated exact sequence variants (ASVs) after error estimation, denoising and chimera removal
- dadasnake quality/error statistics figures and tables for intermediary steps

## How to apply

Build a tab-separated sample table with columns library, r1_file and r2_file, and additionally include run IDs for the raw reads placed in the raw_directory declared in the yaml config. Dadasnake uses these run IDs to group reads so DADA2 estimates a separate error model per run during quality filtering, trimming, error estimation and denoising, with chimera removal afterwards; the DADA2 implementation includes NovaSeq-enabled error models. Set run-specific parameters (e.g. trunc_length such as 90 bp, paired processing) in the human-readable config file, then validate the setup with a dryrun before launching the cluster run with a thread cap (e.g. -t 50). Per-run error models are important when reads from different runs have different quality profiles, since a single pooled model would mis-estimate error rates for some runs and inflate spurious ASVs.

## Related tools

- **dadasnake** (1-command Snakemake pipeline that organizes the sample table run IDs and dispatches the DADA2 error estimation and denoising steps) — https://github.com/a-h-b/dadasnake
- **DADA2** (performs quality filtering and trimming, error estimation & denoising (including NovaSeq-enabled models) and chimera removal) — https://github.com/a-h-b/dadasnake
- **cutadapt** (upstream primer removal and removal of poly-G-tails before error model estimation)
- **Snakemake** (orchestrates the error-estimation and denoising jobs across cluster cores)

## Examples

```
dadasnake -d -r config.yaml
```

## Evaluation signals

- Dryrun ('dadasnake -d -r config.yaml') completes without errors, validating sample table formatting incl. run IDs and config paths
- Per-run error model fit plots produced by dadasnake show converged error rates for each run ID rather than a single pooled curve
- Error/transitions rates decline with quality score in the DADA2 error plots for every run, indicating successful model learning
- Chimera removal and final ASV table generation complete, with outputs handed off in biom format, phyloseq objects, fasta and tab-separated tables
- Workflow finishes with a 'workflow.done' file in the output directory

## Limitations

- Dadasnake has only been tested with Snakemake versions >= 5.9.1; cluster scheduling depends on a supported scheduler (slurm or uge)
- Error models are only as good as the reads per run — very few samples in a run may give unstable DADA2 error estimates
- NovaSeq-enabled models cover NovaSeq-specific error profiles; other unusual platforms may need config adjustment via the human-readable config file

## Evidence

- [methods] error estimation & denoising per-run modeling support: "error estimation & denoising - using DADA2, including Novaseq-enabled models"
- [methods] sample table run IDs enable per-run error models: [evidence span withheld — anchor status: not_found]
- [readme] configurable sample table and steps: "With a human-readable configuration file and a simple sample table, its steps are adjustable to a wide array of input data and requirements."
