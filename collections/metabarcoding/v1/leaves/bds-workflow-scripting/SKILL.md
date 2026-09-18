---
name: bds-workflow-scripting
description: Use when building or customizing a multi-step metabarcoding workflow
  (e.g., PEMA's 4-part 16S/18S/ITS/COI analysis) where users must explore alternative
  tools or parameters per step without complete re-execution.
when_to_use_negative:
- Input is not paired-end Illumina .fastq.gz reads (e.g., already a feature table
  or single-end reads outside the pipeline's design scope)
- The 12S marker gene needs OTU clustering via VSEARCH — PEMA supports 12S only for
  ASV inference with Swarm, not VSEARCH OTUs
- A single fixed algorithm run with no intent to explore alternative tools or parameters
  is needed, so checkpoint-based partial re-execution adds no benefit
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PEMA
  role: The 4-part metabarcoding pipeline implemented as a BDS script whose checkpoint
    files enable partial re-execution
  repo: https://github.com/hariszaf/pema
- name: BDS
  role: BigDataScript programming language providing ad hoc task parallelism, task
    synchronization, roll-back checkpoints, and on-demand partial pipeline execution
  repo: https://pcingola.github.io/BigDataScript/
- name: Docker
  role: Container technology in which the BDS-scripted pipeline is delivered and executed
  repo: https://hub.docker.com/r/hariszaf/pema
- name: Singularity
  role: HPC-centered container alternative for running the BDS-scripted pipeline
  repo: https://github.com/hariszaf/pema/tree/master/singularity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:0c0797ef12b37f826f7677bd90cdfb2bdc4c884eaba9cf63da638b183c0679ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
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
yes_rate: 0.7
n_claims: 10
---

# bds-workflow-scripting

## Summary

Implement a multi-part metabarcoding pipeline as a BigDataScript (BDS) script so that checkpoint files enable partial re-execution and crash recovery, letting users swap algorithms or parameters for individual steps (e.g., OTU clustering vs. ASV inference) without rerunning the whole pipeline.

## When to use

Use when building or customizing a multi-step metabarcoding workflow (e.g., PEMA's 4-part 16S/18S/ITS/COI analysis) where users must explore alternative tools or parameters per step without complete re-execution.

## When NOT to use

- Input is not paired-end Illumina .fastq.gz reads (e.g., already a feature table or single-end reads outside the pipeline's design scope)
- The 12S marker gene needs OTU clustering via VSEARCH — PEMA supports 12S only for ASV inference with Swarm, not VSEARCH OTUs
- A single fixed algorithm run with no intent to explore alternative tools or parameters is needed, so checkpoint-based partial re-execution adds no benefit

## Inputs

- Paired-end .fastq.gz read files from Illumina sequencing (one per sample, in a mydata/ subdirectory)
- parameters.tsv file (version-matched to the PEMA release) specifying per-step tool choices and parameters
- Optional metadata.csv (comma separated) for phyloseq downstream analysis
- Optional phyloseq_in_PEMA.R script customizing downstream ecological analysis

## Outputs

- Checkpoint files per pipeline step enabling partial re-execution
- Per-step subfolders with per-sample result subfolders (e.g., pre-processing, clustering/ASV, taxonomy)
- OTU- or ASV-table with taxonomies of taxa found and their abundances in each sample
- Statistics and a FASTQC diagram of read quality per sample
- Phyloseq-based alpha/beta diversity and taxonomic composition outputs (when requested)

## How to apply

Write the workflow as a BDS script partitioned into self-contained parts: Part 1 wires read pre-processing (FastQC, Trimmomatic, Cutadapt for ITS, BayesHammer, PANDAseq, obiuniq, VSEARCH chimera removal); Part 2 implements (M)OTU clustering with VSEARCH (16S/18S) or CROP with auto-tuned b, e, z parameters (ITS/COI) and Swarm v2 ASV inference with user-configurable singleton removal; Part 3 performs taxonomy assignment via CREST LCAClassifier with Silva 128/132 (16S/18S), Unite (ITS), or RDPClassifier with MIDORI (COI); Part 4 invokes the phyloseq R package for alpha/beta diversity. Emit checkpoint files and per-analysis parameters into distinct self-explanatory subfolders per pipeline step, with per-sample result subfolders, so that changing a downstream step (e.g., switching clustering/ASV algorithms) triggers partial re-execution from the last checkpoint rather than a full rerun. Rationale: BDS's ad hoc task parallelism and task synchronization support heavyweight computation and roll-back checkpoints, addressing the inability of existing pipelines to partially re-execute an analysis. Containerize the resulting script with Docker and a corresponding Singularity file for portability across HPC and personal computers.

## Related tools

- **PEMA** (The 4-part metabarcoding pipeline implemented as a BDS script whose checkpoint files enable partial re-execution) — https://github.com/hariszaf/pema
- **BDS** (BigDataScript programming language providing ad hoc task parallelism, task synchronization, roll-back checkpoints, and on-demand partial pipeline execution) — https://pcingola.github.io/BigDataScript/
- **Docker** (Container technology in which the BDS-scripted pipeline is delivered and executed) — https://hub.docker.com/r/hariszaf/pema
- **Singularity** (HPC-centered container alternative for running the BDS-scripted pipeline) — https://github.com/hariszaf/pema/tree/master/singularity

## Examples

```
./pema_latest.bds
```

## Evaluation signals

- Running the pipeline end-to-end on the tutorial 'analysis_directory' test data executes all 4 parts of the BDS script
- Checkpoint files are emitted after each pipeline step into distinct self-explanatory subfolders
- Switching a clustering/ASV algorithm (e.g., VSEARCH to Swarm) triggers partial re-execution from the relevant checkpoint rather than a complete rerun
- The final (M)OTU/ASV table with taxonomy and abundances is returned alongside per-sample FASTQC quality diagrams
- If PEMA runs on a laptop with 4 cores, it uses all four — confirming BDS task parallelism is active

## Limitations

- COI taxonomy assignment requires a minimum of 20 GB of RAM
- The parameters.tsv file version must exactly match the PEMA version being used, and mandatory files/directories must be named exactly as specified or the run fails
- Running on a personal computer is significantly slower than on an HPC environment; a common PC only suffices for analyses with a small number of samples

## Evidence

- [abstract] Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution.: "Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution."
- [intro] It is designed for paired-end sequencing studies and is implemented in the BDS [11] programming language: "It is designed for paired-end sequencing studies and is implemented in the BDS [11] programming language"
- [readme] PEMA has been implemented in BigDataScript programming language. BDS’s ad hoc task parallelism and task synchronization, supports heavyweight computation.: [evidence span withheld — anchor status: not_found]
- [readme] Thus, PEMA inherits such features and it also supports roll-back checkpoints and on-demand partial pipeline execution.: [evidence span withheld — anchor status: not_found]
- [intro] Among the most prominent are technical difficulties in installation and use, strict limitations in setting parameters for the algorithms invoked, and incompetence in partial re-execution of an: "Among the most prominent are technical difficulties in installation and use, strict limitations in setting parameters for the algorithms invoked, and incompetence in partial re-execution of an"
