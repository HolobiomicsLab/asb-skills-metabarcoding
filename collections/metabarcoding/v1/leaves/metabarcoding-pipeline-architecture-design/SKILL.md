---
name: metabarcoding-pipeline-architecture-design
description: Use when building or extending a metabarcoding pipeline for 16S/18S/ITS/COI
  marker genes that must support tool/parameter exploration without re-running earlier
  stages, e.g. swapping OTU clustering or ASV inference methods.
when_to_use_negative:
- The input is already a processed feature/OTU table — the pipeline expects raw .fastq.gz
  reads.
- Only a single fixed analysis with no parameter exploration is needed and partial
  re-execution adds no value.
- The analysis targets 12S rRNA OTUs via VSEARCH — this combination is not supported
  (12S supports only ASV/Swarm inference).
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: PEMA
  role: the pipeline being architected; bundles all metabarcoding analysis steps
  repo: https://github.com/hariszaf/pema
- name: BDS
  role: BigDataScript language providing task parallelism, roll-back checkpoints and
    on-demand partial pipeline execution
- name: Docker
  role: containerization technology making PEMA accessible across operating systems
  repo: https://hub.docker.com/r/hariszaf/pema
- name: VSEARCH
  role: OTU clustering and chimera removal
- name: Swarm v2
  role: ASV inference for all marker genes
- name: phyloseq
  role: R package invoked in Part 4 for alpha/beta diversity and taxonomic composition
  repo: http://joey711.github.io/phyloseq/index.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:30c18c1a50c19591cb06d05e2b3079f866f28bd66b9c3274cf44b232b0d045d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - Docker
  - PEMA
  - Swarm v2
  - VSEARCH
  - phyloseq
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabarcoding-pipeline-architecture-design

## Summary

Design of a containerized, 4-part eDNA metabarcoding pipeline (PEMA) implemented in BigDataScript so that checkpoint files enable partial re-execution when users explore alternative algorithms or parameters for specific steps.

## When to use

Use when building or extending a metabarcoding pipeline for 16S/18S/ITS/COI marker genes that must support tool/parameter exploration without re-running earlier stages, e.g. swapping OTU clustering or ASV inference methods.

## When NOT to use

- The input is already a processed feature/OTU table — the pipeline expects raw .fastq.gz reads.
- Only a single fixed analysis with no parameter exploration is needed and partial re-execution adds no value.
- The analysis targets 12S rRNA OTUs via VSEARCH — this combination is not supported (12S supports only ASV/Swarm inference).

## Inputs

- .fastq.gz paired-end read files (Illumina) per sample, placed in a 'mydata' subdirectory
- parameters.tsv (version-matched to the PEMA release) in the analysis directory
- optional metadata.csv (comma-separated) and phyloseq_in_PEMA.R for downstream analysis

## Outputs

- OTU- or ASV-table with taxonomies and per-sample abundances
- checkpoint files and per-analysis parameters in per-step, per-sample subfolders enabling partial re-execution
- per-sample quality statistics and FASTQC quality diagrams
- phyloseq-based alpha/beta diversity and taxonomic composition outputs when downstream analysis is requested

## How to apply

Structure the pipeline in BigDataScript (BDS) as four parts: (1) read pre-processing wiring FastQC, Trimmomatic, Cutadapt (ITS only), BayesHammer, PANDAseq, obiuniq (OBITools) and VSEARCH chimera removal; (2) OTU clustering with VSEARCH (16S/18S) or CROP (ITS/COI, auto-tuned b, e, z parameters) and ASV inference with Swarm v2 (for Swarm v2, defer chimera removal until after ASV inference), plus user-configurable singleton removal; (3) taxonomy assignment via CREST LCAClassifier with Silva 128/132 and Unite for 16S/18S/ITS, RDPClassifier with MIDORI for COI, and an optional phylogeny-based route for 16S rRNA using a 1,000-sequence Silva-derived RAxML-ng/gappa reference tree with PaPaRa and EPA-ng; (4) downstream analysis via the phyloseq R package with optional MAFFT alignment and RAxML-ng tree building. Exploit BDS's roll-back checkpoints and on-demand partial pipeline execution: emit checkpoint files and per-analysis parameters into distinct self-explanatory subfolders per pipeline step (with per-sample result subfolders) so switching clustering/ASV algorithms triggers only partial re-execution. Parameterize everything through a version-matched parameters.tsv file in the analysis directory, and containerize with Docker and Singularity for OS portability and HPC use.

## Related tools

- **PEMA** (the pipeline being architected; bundles all metabarcoding analysis steps) — https://github.com/hariszaf/pema
- **BDS** (BigDataScript language providing task parallelism, roll-back checkpoints and on-demand partial pipeline execution)
- **Docker** (containerization technology making PEMA accessible across operating systems) — https://hub.docker.com/r/hariszaf/pema
- **VSEARCH** (OTU clustering and chimera removal)
- **Swarm v2** (ASV inference for all marker genes)
- **phyloseq** (R package invoked in Part 4 for alpha/beta diversity and taxonomic composition) — http://joey711.github.io/phyloseq/index.html

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- Run the pipeline end-to-end on the tutorial 'analysis_directory' test data and confirm all 4 BDS parts execute and a final (M)OTU/ASV table with taxonomy and abundances is returned.
- Verify checkpoint files are produced in distinct per-step, per-sample subfolders after each pipeline part.
- Change a clustering/ASV algorithm or parameter in parameters.tsv and confirm only downstream steps re-execute (partial re-execution) rather than a full rerun.
- Confirm the analysis directory layout matches the mandatory structure (mydata/, parameters.tsv, and version-matched parameters file for the PEMA release in use).
- Confirm the Docker/Singularity container mounts the analysis directory and outputs are written back to the mounted path.

## Limitations

- COI taxonomy assignment requires a minimum of 20 GB of RAM.
- The parameters.tsv file must match the exact PEMA version being used, and files/directories must be named exactly as specified or PEMA fails.
- Running on a simple PC is possible but slow; HPC environments are recommended to exploit BDS task parallelism and available cores.
- Third-party tool licenses are separate (see Additional File 1); PEMA itself is GNU GPLv3.

## Evidence

- [abstract] Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution.: "Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution."
- [readme] BDS's ad hoc task parallelism and task synchronization, supports heavyweight computation. Thus, PEMA inherits such features and it also supports roll-back checkpoints and on-demand partial pipeline execution.: [evidence span withheld — anchor status: not_found]
- [intro] Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs: "Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs"
- [readme] For COI, a minimum of 20 GB of RAM for the taxonomy assignment step is required.: [evidence span withheld — anchor status: not_found]
