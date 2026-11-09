---
name: its-mock-community-metabarcoding-analysis
description: Use when evaluating an ITS (fungal) metabarcoding pipeline against a
  mock community of known composition, e.g. the 19-taxon Bakker ITS2 .fastq read pairs
  from ENA, to compare recovered taxa, false positives, and genus-level detection.
when_to_use_negative:
- Input data are 16S/18S rRNA or COI amplicons — the Unite database and this ITS-specific
  parameter setting apply only to the ITS marker gene.
- The goal is OTU clustering with VSEARCH rather than ASV inference — this skill specifically
  validates the Swarm v2 (d=20) ASV configuration.
- Data are whole metagenome shotgun reads rather than Illumina amplicon .fastq.gz
  pairs — PEMA accepts amplicon .fastq.gz from Illumina platforms.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: PEMA
  role: Containerized pipeline orchestrating the full ITS metabarcoding analysis from
    raw reads to assigned ASV table
  repo: https://github.com/hariszaf/pema
- name: FastQC
  role: Per-sample read-quality summaries of the raw ITS .fastq pairs
- name: Trimmomatic
  role: Adapter/primer removal and quality trimming of ITS reads
- name: Cutadapt
  role: Handles the length variability of the ITS marker during primer removal
- name: BayesHammer (SPAdes)
  role: Base error correction of the trimmed reads
- name: PANDAseq
  role: Assembles overlapping paired-end ITS reads
- name: OBITools (obiuniq)
  role: Dereplicates identical sequences per sample keeping abundance counts
- name: Swarm v2
  role: ASV inference with cluster radius d = 20 targeting the ITS2 region
- name: CREST LCAClassifier
  role: Alignment-based taxonomy assignment of ASVs against Unite
- name: Unite database
  role: Reference database for ITS taxonomy assignment
- name: Docker
  role: Container runtime to execute PEMA with the analysis directory mounted
  repo: https://hub.docker.com/r/hariszaf/pema
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:7f9e629336cce87f06374fa0ee33941134c108897cc6d9e94a190faaa0589e37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - BayesHammer (SPAdes)
  - CREST LCAClassifier
  - CROP
  - Cutadapt
  - Docker
  - FastQC
  - OBITools
  - OBITools (obiuniq)
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - Unite database
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# its-mock-community-metabarcoding-analysis

## Summary

Reproduces the PEMA ITS fungal mock community evaluation (Bakker dataset, 19 fungal taxa) by running the containerized PEMA pipeline — Trimmomatic/Cutadapt pre-processing, BayesHammer error correction, PANDAseq assembly, Swarm v2 ASV inference (d=20), and CREST LCAClassifier taxonomy against Unite — then validating recovery of expected taxa.

## When to use

Use when evaluating an ITS (fungal) metabarcoding pipeline against a mock community of known composition, e.g. the 19-taxon Bakker ITS2 .fastq read pairs from ENA, to compare recovered taxa, false positives, and genus-level detection.

## When NOT to use

- Input data are 16S/18S rRNA or COI amplicons — the Unite database and this ITS-specific parameter setting apply only to the ITS marker gene.
- The goal is OTU clustering with VSEARCH rather than ASV inference — this skill specifically validates the Swarm v2 (d=20) ASV configuration.
- Data are whole metagenome shotgun reads rather than Illumina amplicon .fastq.gz pairs — PEMA accepts amplicon .fastq.gz from Illumina platforms.

## Inputs

- ITS mock community paired-end .fastq/.fastq.gz reads (Bakker dataset, 19 fungal taxa, 3 samples) from ENA
- PEMA parameters.tsv configured for the ITS marker gene (Swarm v2, d=20, Unite database)
- Unite reference database for ITS taxonomy assignment

## Outputs

- ASV table with taxonomically assigned fungal taxa and per-sample abundances
- Per-sample FastQC read-quality summaries/diagrams
- Taxonomic assignment report (expected: Chytriomyces in all 3 samples; Fusarium resolved only to Nectriaceae; ≤1 false-positive assignment)

## How to apply

Obtain the ITS mock community .fastq read pairs of Bakker (19 fungal taxa, 3 samples) from the European Nucleotide Archive and inspect per-sample quality with FastQC. Pre-process reads with Trimmomatic (adapter/primer removal, quality trimming) and Cutadapt to handle the length variability of the ITS marker, error-correct bases with BayesHammer (SPAdes), assemble overlapping paired-end reads with PANDAseq, and dereplicate identical sequences per sample with obiuniq (OBITools). Infer ASVs with Swarm v2 using cluster radius d = 20 targeting the ITS2 region; Swarm v2 is not affected by chimeras, so chimera removal is performed after ASV inference. Assign taxonomy with the alignment-based CREST LCAClassifier against the Unite reference database, then compile the per-sample abundance table and evaluate recovery: Chytriomyces should appear in all 3 samples, Fusarium should not be resolved beyond its family (Nectriaceae), and false-positive assignments should not exceed the single FP reported for this parameter setting.

## Related tools

- **PEMA** (Containerized pipeline orchestrating the full ITS metabarcoding analysis from raw reads to assigned ASV table) — https://github.com/hariszaf/pema
- **FastQC** (Per-sample read-quality summaries of the raw ITS .fastq pairs)
- **Trimmomatic** (Adapter/primer removal and quality trimming of ITS reads)
- **Cutadapt** (Handles the length variability of the ITS marker during primer removal)
- **BayesHammer (SPAdes)** (Base error correction of the trimmed reads)
- **PANDAseq** (Assembles overlapping paired-end ITS reads)
- **OBITools (obiuniq)** (Dereplicates identical sequences per sample keeping abundance counts)
- **Swarm v2** (ASV inference with cluster radius d = 20 targeting the ITS2 region)
- **CREST LCAClassifier** (Alignment-based taxonomy assignment of ASVs against Unite)
- **Unite database** (Reference database for ITS taxonomy assignment)
- **Docker** (Container runtime to execute PEMA with the analysis directory mounted) — https://hub.docker.com/r/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema:v.2.1.4 ./pema_latest.bds
```

## Evaluation signals

- The ASV table exists with assigned taxonomies and per-sample abundances.
- The genus Chytriomyces is detected in all 3 samples.
- Fusarium species are not resolved beyond their family (Nectriaceae).
- False-positive taxonomic assignments do not exceed the single FP reported for this parameter setting.
- FastQC outputs show per-sample read-quality summaries consistent with the raw input .fastq pairs.

## Limitations

- Fusarium cannot be resolved beyond family level (Nectriaceae) with this parameter setting and the Unite database.
- Many parameter settings had to be tested, especially for quality trimming and OTU clustering/ASV inference, so results are parameter-setting dependent.
- Running on a personal PC can be slow; HPC is recommended, and COI taxonomy assignment requires ≥20 GB RAM (not needed here but constrains container sizing for multi-marker runs).

## Evidence

- [methods] Swarm v2 ASV inference with d=20 and post-inference chimera removal: "Swarm v2 [19] allows ASV inference in all cases"
- [methods] obiuniq dereplication of identical sequences per sample: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
- [methods] PANDAseq assembles overlapping paired-end reads: "PANDAseq [34] assembles the overlapping paired-end reads"
- [abstract] PEMA was evaluated against mock communities and published datasets with comparable quality: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [intro] Unite database is used for the ITS gene taxonomy assignment: "the Unite database [22] is used for the ITS gene"
- [readme] PEMA accepts Illumina .fastq.gz and returns OTU/ASV tables with taxonomies and abundances: [evidence span withheld — anchor status: not_found]
