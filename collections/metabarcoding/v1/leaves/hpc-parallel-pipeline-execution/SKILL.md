---
name: hpc-parallel-pipeline-execution
description: Use when raw .fastq.gz amplicon reads (16S/18S/ITS/COI/12S) must be processed
  end-to-end and the computing environment is an HPC server or cluster where multi-core
  parallelization and checkpointing pay off.
when_to_use_negative:
- Analyses with a very small number of samples where a common PC suffices — the HPC/parallel
  route adds setup overhead without benefit
- COI analyses on machines with less than 20 GB of RAM, since the taxonomy assignment
  step will not run
- Runs where the parameters.tsv version does not match the PEMA image version (e.g.,
  v.2.1.4 image with an older parameters file) — PEMA will fail
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0632
tools:
- name: PEMA
  role: Containerized metabarcoding pipeline executed on HPC nodes; reads pre-processing,
    OTU clustering/ASV inference, and taxonomy assignment
  repo: https://github.com/hariszaf/pema
- name: BDS
  role: BigDataScript language implementing PEMA; provides ad hoc task parallelism,
    task synchronization, roll-back checkpoints, and partial pipeline execution across
    all cores
- name: Docker
  role: Container runtime alternative (image hariszaf/pema:v.2.1.4) for launching
    PEMA with the analysis directory mounted
  repo: https://hub.docker.com/r/hariszaf/pema
- name: Singularity
  role: HPC-centered container runtime; PEMA image pulled from Singularity Hub and
    run with the analysis directory bind-mounted
  repo: https://singularity-hub.org/collections/2295
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:3a033bf42793f71e4167e2acb67a820a0b01e9f861bc208132df0f63ff6eac1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - Docker
  - LotuS
  - PEMA
  - QIIME 2
  - Singularity
  - mothur
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# hpc-parallel-pipeline-execution

## Summary

Run the PEMA metabarcoding pipeline on HPC nodes, exploiting BigDataScript (BDS) intrinsic parallelization to distribute pipeline tasks across all available cores (e.g., 20 Intel Xeon 2.60 GHz cores on Dell M630 nodes with 128 GB RAM) instead of single-threaded execution on a PC.

## When to use

Use when raw .fastq.gz amplicon reads (16S/18S/ITS/COI/12S) must be processed end-to-end and the computing environment is an HPC server or cluster where multi-core parallelization and checkpointing pay off.

## When NOT to use

- Analyses with a very small number of samples where a common PC suffices — the HPC/parallel route adds setup overhead without benefit
- COI analyses on machines with less than 20 GB of RAM, since the taxonomy assignment step will not run
- Runs where the parameters.tsv version does not match the PEMA image version (e.g., v.2.1.4 image with an older parameters file) — PEMA will fail

## Inputs

- .fastq.gz paired-end Illumina read files in the mydata/ subdirectory
- parameters.tsv matched to the PEMA version being run (e.g., v.2.1.4)
- metadata.csv (comma-separated) for downstream phyloseq analysis
- phyloseq_in_PEMA.R R script (optional, for downstream ecological analysis)

## Outputs

- OTU- or ASV-table with taxonomies and per-sample abundances
- Per-sample read-quality statistics and FASTQC diagrams
- Downstream phyloseq ecological analysis outputs (alpha/beta diversity, taxonomic composition)

## How to apply

Prepare an analysis directory containing the version-matched parameters.tsv, a mydata/ subdirectory with the .fastq.gz reads, and optionally phyloseq_in_PEMA.R plus a comma-separated metadata.csv. Pull the container (Docker: hariszaf/pema:v.2.1.4, or the Singularity pema_v.2.1.4.sif image) and mount the analysis directory under /mnt/analysis. Launch PEMA via ./pema_latest.bds inside the container; BDS's ad hoc task parallelism and task synchronization automatically exploit all cores of the machine, and roll-back checkpoints allow on-demand partial re-execution without rerunning the whole pipeline. Rationale: the article ran PEMA on Dell M630 HPC nodes (128 GB RAM, 20 Intel Xeon 2.60 GHz cores) using BDS parallelization to process the Pavloudi et al. 16S salinity-gradient reads, while noting that a common PC can suffice only for small sample numbers and that COI taxonomy assignment requires at least 20 GB RAM.

## Related tools

- **PEMA** (Containerized metabarcoding pipeline executed on HPC nodes; reads pre-processing, OTU clustering/ASV inference, and taxonomy assignment) — https://github.com/hariszaf/pema
- **BDS** (BigDataScript language implementing PEMA; provides ad hoc task parallelism, task synchronization, roll-back checkpoints, and partial pipeline execution across all cores)
- **Docker** (Container runtime alternative (image hariszaf/pema:v.2.1.4) for launching PEMA with the analysis directory mounted) — https://hub.docker.com/r/hariszaf/pema
- **Singularity** (HPC-centered container runtime; PEMA image pulled from Singularity Hub and run with the analysis directory bind-mounted) — https://singularity-hub.org/collections/2295

## Examples

```
singularity run -B /<path_to_analysis_directory>/:/mnt/analysis pema_v.2.1.4.sif
```

## Evaluation signals

- The analysis directory contains exactly the mandatory files/directories named as PEMA requires: mydata, parameters.tsv (and phyloseq_in_PEMA.R, metadata.csv if phyloseq is requested) — PEMA fails otherwise
- parameters.tsv version matches the PEMA container version (e.g., parameters file v.2.1.4 for pema:v.2.1.4)
- On a 20-core node, BDS parallelizes across all 20 Intel Xeon 2.60 GHz cores as in the article's Pavloudi et al. run
- Output OTU/ASV tables include taxonomies and per-sample abundances plus per-sample FASTQC quality diagrams and statistics
- Checkpoint/partial re-execution works: modifying a single pipeline step does not force complete re-execution

## Limitations

- Runtime depends on the computational features of the environment, data size, and chosen parameters; a common PC is only adequate for analyses with a small number of samples
- COI taxonomy assignment requires a minimum of 20 GB of RAM
- PEMA fails unless files and directories are named exactly as prescribed (mydata, parameters.tsv, phyloseq_in_PEMA.R, metadata.csv)
- The parameters file must match the PEMA version being used, or the run will not proceed correctly

## Evidence

- [readme] BDS's ad hoc task parallelism and task synchronization, supports heavyweight computation. Thus, PEMA inherits such features and it also supports roll-back checkpoints and on-demand partial pipeline execution.: [evidence span withheld — anchor status: not_found]
- [methods] Run PEMA on the dataset on Dell M630 HPC nodes (128 GB RAM, 20 Intel Xeon 2.60 GHz cores), exploiting BDS intrinsic parallelization across all 20 cores.: [evidence span withheld — anchor status: not_found]
- [readme] For COI, a minimum of 20 GB of RAM for the taxonomy assignment step is required.: [evidence span withheld — anchor status: not_found]
- [readme] if PEMA is executed on a personal laptop with 4 cores, it is going to use all four of them.: [evidence span withheld — anchor status: not_found]
- [abstract] A high-performance computing-based approach was used to develop PEMA; however, it can be used in personal computers as well.: "A high-performance computing-based approach was used to develop PEMA; however, it can be used in personal computers as well."
