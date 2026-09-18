---
name: docker-singularity-containerization
description: Use when a pipeline bundles many current tools that are technically difficult
  to install (e.g., PEMA's metabarcoding stack), and portability across HPC clusters
  (Singularity) and personal PCs (Docker) is needed.
when_to_use_negative:
- Only a single tool from the pipeline is needed and it is trivially installable natively
  on the host
- The execution environment cannot mount host directories or run privileged container
  runtimes (no Docker/Singularity support)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Docker
  role: Container runtime used to build, distribute (Docker Hub) and run PEMA on PCs
    and servers
  repo: https://hub.docker.com/r/hariszaf/pema
- name: Singularity
  role: HPC-centered container technology; PEMA's corresponding Singularity file/image
    enables execution on clusters
  repo: https://github.com/hariszaf/pema/tree/master/singularity
- name: PEMA
  role: The containerized metabarcoding pipeline being deployed; entrypoint pema_latest.bds
    executed inside the container
  repo: https://github.com/hariszaf/pema
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:72f78451f9d7ca2f38df3dc1607f4fd23d40cbc9a1cdfadd607ef029fe9342c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BDS
  - Docker
  - PEMA
  - Singularity
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# docker-singularity-containerization

## Summary

Package a multi-tool bioinformatics pipeline (here PEMA, implemented in BigDataScript) as Docker and Singularity container images so it can be deployed with low installation effort on both HPC environments and personal computers across operating systems.

## When to use

Use when a pipeline bundles many state-of-the-art tools that are technically difficult to install (e.g., PEMA's metabarcoding stack), and portability across HPC clusters (Singularity) and personal PCs (Docker) is needed.

## When NOT to use

- Only a single tool from the pipeline is needed and it is trivially installable natively on the host
- The execution environment cannot mount host directories or run privileged container runtimes (no Docker/Singularity support)

## Inputs

- Dockerfile / Singularity definition file for the pipeline
- analysis_directory containing parameters.tsv and a mydata/ subdirectory with .fastq.gz read files
- optional phyloseq_in_PEMA.R script and metadata.csv (comma-separated) for downstream analysis

## Outputs

- Runnable Docker image (e.g., hariszaf/pema:v.2.1.4 on Docker Hub)
- Runnable Singularity image (.sif file, e.g., pema_v.2.1.4.sif)
- Pipeline outputs (OTU/ASV table with taxonomies and abundances) written to the mounted analysis directory on the host

## How to apply

Build the pipeline's Docker image and produce the corresponding Singularity file (e.g., singularity pull of pema_v.2.1.4.sif from the GitLab-hosted image or docker pull hariszaf/pema:v.2.1.4). Run the container with the user's analysis directory mounted at /mnt/analysis (Docker -v flag or Singularity -B bind flag) so all inputs and outputs persist on the host. Inside the container, launch the BDS entrypoint ./pema_latest.bds; the container makes PEMA easy accessible for all operating systems without manual tool installation. Always match the parameters.tsv version to the PEMA image version being used (e.g., v.2.1.4 parameters file for pema:v.2.1.4). Prefer HPC environments to exploit the full potential of the pipeline, though a common PC can suffice for analyses with a small number of samples.

## Related tools

- **Docker** (Container runtime used to build, distribute (Docker Hub) and run PEMA on PCs and servers) — https://hub.docker.com/r/hariszaf/pema
- **Singularity** (HPC-centered container technology; PEMA's corresponding Singularity file/image enables execution on clusters) — https://github.com/hariszaf/pema/tree/master/singularity
- **PEMA** (The containerized metabarcoding pipeline being deployed; entrypoint pema_latest.bds executed inside the container) — https://github.com/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema
```

## Evaluation signals

- The image pull succeeds: `docker pull hariszaf/pema:v.2.1.4` or `singularity pull pema_v.2.1.4.sif` completes without errors
- The container initiates with the analysis directory mounted under /mnt/analysis and the host directory is visible inside the container
- Running ./pema_latest.bds inside the container executes the full BDS pipeline and all output files land in the mounted host directory
- The parameters.tsv version matches the PEMA image version (v.2.1.4 parameters file with pema:v.2.1.4), otherwise the run fails
- Containerized runs reproduce results of comparable quality to non-containerized execution, e.g., mock community F1-scores in the 74%-86% range reported for PEMA

## Limitations

- PEMA will fail unless mandatory files and directories are named exactly as specified (parameters.tsv, mydata, and optionally phyloseq_in_PEMA.R and metadata.csv)
- For the COI marker gene, a minimum of 20 GB of RAM is required for the taxonomy assignment step
- Running on a common PC is only sufficient for analyses with a small number of samples; HPC environments are suggested to exploit the full potential of PEMA

## Evidence

- [intro] via software containerization technologies such as Docker [12] and Singularity [13], with the latter being HPC-centered: "via software containerization technologies such as Docker [12] and Singularity [13], with the latter being HPC-centered"
- [readme] container-based technologies such as Docker and Singularity make PEMA easy accessible: [evidence span withheld — anchor status: not_found]
- [readme] PEMA runs either as a Docker or as a Singularity image: [evidence span withheld — anchor status: not_found]
- [abstract] PEMA is a containerized assembly of key metabarcoding analysis tools that requires low effort in setting up: "PEMA is a containerized assembly of key metabarcoding analysis tools that requires low effort in setting up, running, and customizing to researchers' needs."
