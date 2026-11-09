---
name: dada2-asv-denoising-and-pair-merging
description: Use when raw paired-end amplicon FASTQ reads (e.g. Illumina 16S with
  515F/806R primers) must be denoised into exact ASVs with error modeling, pair merging
  and chimera removal, as in the prokaryotic mock community benchmark (ERR777696).
when_to_use_negative:
- Input is already an ASV/OTU count table — denoising and merging have already been
  done
- Data are long-read amplicons or single-read runs where DADA2 paired merging with
  min_overlap 12 does not apply
- The goal is taxonomic novelty discovery rather than exact variant resolution — classic
  OTU clustering may be preferred
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: dadasnake
  role: 1-command Snakemake pipeline wrapping DADA2 error estimation, denoising, pair
    merging and chimera removal for amplicon FASTQ input
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: performs quality filtering and trimming, error estimation and denoising, pair
    merging, and consensus chimera removal
- name: seqtk
  role: optional downsampling of reads per sample before denoising
  repo: https://github.com/lh3/seqtk
- name: cutadapt
  role: removes primers and poly-G-tails prior to DADA2 denoising (min overlap 10,
    0.2 mismatch rate)
- name: Snakemake
  role: drives the DADA2 steps as multiple jobs on a computing cluster from a single
    conda environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:a76bc410337aadcb6693dd12c01f68d4000a054f431e2bf578eeb3acb24a8177
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - DADA2
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  - seqtk
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# dada2-asv-denoising-and-pair-merging

## Summary

Delineate exact sequence variants (ASVs) from quality-filtered amplicon reads using the DADA2 algorithm — error estimation, denoising, paired-end merging (min_overlap 12, max_mismatch 0) and consensus chimera removal — as wrapped by the dadasnake Snakemake pipeline.

## When to use

Use when raw paired-end amplicon FASTQ reads (e.g. Illumina 16S with 515F/806R primers) must be denoised into exact ASVs with error modeling, pair merging and chimera removal, as in the prokaryotic mock community benchmark (ERR777696).

## When NOT to use

- Input is already an ASV/OTU count table — denoising and merging have already been done
- Data are long-read amplicons or single-read runs where DADA2 paired merging with min_overlap 12 does not apply
- The goal is taxonomic novelty discovery rather than exact variant resolution — classic OTU clustering may be preferred

## Inputs

- paired-end amplicon FASTQ files (e.g. ERR777696 for the 59-species prokaryotic mock community)
- tab-separated sample table with library/r1_file/r2_file columns
- dadasnake YAML configuration file (paired=true, raw_directory, database paths)

## Outputs

- ASV sequence set and count table (tab-separated hand-off files from the dadasnake output directory)
- intermediary quality profiles and step statistics reported in figures and tables

## How to apply

After primer removal with cutadapt (e.g. 515F GTGYCAGCMGCCGCGGTAA and 806R GGACTACNVGGGTWTCTAAT, minimum overlap 10, max 2 primers per end, 0.2 mismatch rate), run DADA2 via the dadasnake Snakemake workflow: quality filter and trim reads, estimate errors (including Novaseq-enabled models), denoise, merge paired reads with min_overlap 12 and max_mismatch 0, and remove chimeras with the consensus method. Optionally downsample reads per sample with seqtk and check ASV lengths in R. The expected result for the 59-species bacterial/archaeal mock community (ENA accession ERR777696) is 59 prokaryotic ASVs, ignoring the four known contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas). Snakemake drives the steps as separate cluster jobs inside a single conda environment, and dadasnake reports intermediary statistics in figures and tables.

## Related tools

- **dadasnake** (1-command Snakemake pipeline wrapping DADA2 error estimation, denoising, pair merging and chimera removal for amplicon FASTQ input) — https://github.com/a-h-b/dadasnake
- **DADA2** (performs quality filtering and trimming, error estimation and denoising, pair merging, and consensus chimera removal)
- **seqtk** (optional downsampling of reads per sample before denoising) — https://github.com/lh3/seqtk
- **cutadapt** (removes primers and poly-G-tails prior to DADA2 denoising (min overlap 10, 0.2 mismatch rate))
- **Snakemake** (drives the DADA2 steps as multiple jobs on a computing cluster from a single conda environment)

## Examples

```
git clone https://github.com/a-h-b/dadasnake.git && cd dadasnake
```

## Evaluation signals

- Mock community recovery: run on the 59-species bacterial/archaeal mock (ERR777696) yields exactly 59 prokaryotic ASVs (ignoring contaminants)
- Genus-level composition of inferred ASVs matches the ground-truth mock composition after accounting for the known contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas)
- Pair merging succeeded with the default parameters (min_overlap 12, max_mismatch 0) — reads not meeting overlap constraints are reported in dadasnake's intermediary statistics
- Chimera removal by the consensus method leaves no flagged chimera sequences in the final ASV table
- Final ASV/OTU tables pass dadasnake's final_table_filtering and are exported as valid tab-separated hand-off files

## Limitations

- Parameters assume Illumina paired-end amplicon data; reads must retain enough overlap for merging at min_overlap 12 with zero mismatches
- Downsampling via seqtk is optional and reduces depth, which can affect detection of low-abundance ASVs
- The workflow targets microbial ecology amplicon processing on cluster infrastructures (Slurm/UGE); settings in the cluster config (e.g. slurm.config.yaml, MAX_THREADS default 50) must match the local environment

## Evidence

- [abstract] We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants"
- [methods] error estimation & denoising - using DADA2, including Novaseq-enabled models: "error estimation & denoising - using DADA2, including Novaseq-enabled models"
- [methods] chimera removal - using DADA2: "chimera removal - using DADA2"
- [intro] dadasnake reports on intermediary steps and statistics in intuitive figures and tables: "dadasnake reports on intermediary steps and statistics in intuitive figures and tables"
- [readme] It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake.: "It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake."
