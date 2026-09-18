---
name: amplicon-read-quality-preprocessing
description: Use when you have raw paired-end .fastq.gz amplicon reads from Illumina
  sequencing for 16S/18S rRNA, ITS, COI, or 12S rRNA marker genes and need QC, trimming,
  error correction, assembly, dereplication, and chimera removal before OTU/ASV analysis.
when_to_use_negative:
- Input reads are already trimmed, assembled, and dereplicated — PEMA's Part 1 expects
  raw .fastq.gz Illumina reads and will re-run trimming/assembly redundantly.
- The analysis is single-end or produced by a platform other than Illumina — PEMA
  is designed for paired-end sequencing studies with .fastq.gz input.
- You plan to switch tools within Part 1 itself rather than Part 2 — checkpoints mainly
  let you re-run downstream clustering/ASV/taxonomy parts without repeating pre-processing,
  not to swap trimming algorithms mid-run.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: FastQC
  role: Produces quality summary and FASTQC diagram of read quality for each sample
    before trimming
  repo: https://github.com/hariszaf/pema
- name: Trimmomatic
  role: Removes adapters, primers, and performs quality trimming of raw reads (parameterized
    per marker gene)
  repo: https://github.com/hariszaf/pema
- name: Cutadapt
  role: Invoked for primer removal in the ITS marker gene workflow
  repo: https://github.com/hariszaf/pema
- name: BayesHammer (SPAdes)
  role: Base error correction on trimmed reads before paired-end assembly
  repo: https://github.com/hariszaf/pema
- name: PANDAseq
  role: Assembles overlapping paired-end reads after error correction
  repo: https://github.com/hariszaf/pema
- name: obiuniq (OBITools)
  role: Groups all identical sequences per sample and keeps track of their abundances
  repo: https://github.com/hariszaf/pema
- name: VSEARCH
  role: Chimera removal after dereplication (deferred to post-ASV-inference when Swarm
    v2 is used)
  repo: https://github.com/hariszaf/pema
- name: PEMA
  role: Containerized pipeline wrapper (Docker/Singularity image, BDS script) orchestrating
    all Part 1 steps with checkpointing
  repo: https://github.com/hariszaf/pema
merged_aliases:
- amplicon-read-preprocessing-orchestration
merged_alias_records:
- alias: amplicon-read-preprocessing-orchestration
  slug: amplicon-read-preprocessing-orchestration
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:39aab87011259145c7d8e26503418392824228f78adfe9b7415f03e714520826
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - BayesHammer
  - BayesHammer (SPAdes)
  - CREST LCAClassifier
  - Cutadapt
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Trimmomatic
  - VSEARCH
  - obiuniq (OBITools)
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# amplicon-read-quality-preprocessing

## Summary

PEMA's Part 1 read pre-processing stage for paired-end Illumina metabarcoding .fastq.gz files: FastQC quality summaries, Trimmomatic adapter/primer/quality trimming, BayesHammer (SPAdes) base-error correction, PANDAseq paired-end assembly, obiuniq (OBITools) dereplication, and VSEARCH chimera removal before OTU clustering or ASV inference.

## When to use

Use when you have raw paired-end .fastq.gz amplicon reads from Illumina sequencing for 16S/18S rRNA, ITS, COI, or 12S rRNA marker genes and need QC, trimming, error correction, assembly, dereplication, and chimera removal before OTU/ASV analysis.

## When NOT to use

- Input reads are already trimmed, assembled, and dereplicated — PEMA's Part 1 expects raw .fastq.gz Illumina reads and will re-run trimming/assembly redundantly.
- The analysis is single-end or produced by a platform other than Illumina — PEMA is designed for paired-end sequencing studies with .fastq.gz input.
- You plan to switch tools within Part 1 itself rather than Part 2 — checkpoints mainly let you re-run downstream clustering/ASV/taxonomy parts without repeating pre-processing, not to swap trimming algorithms mid-run.

## Inputs

- Paired-end Illumina .fastq.gz files per sample (input format required by PEMA)
- parameters.tsv file specifying the marker gene (16S/18S/12S/ITS/COI) and tool parameters
- Primer/adapter sequences for trimming (via Trimmomatic parameters)
- Analysis directory containing 'mydata' subdirectory and parameters.tsv

## Outputs

- Dereplicated, chimera-checked sequence sets with per-sample abundances (obiuniq output)
- Quality summary statistics per sample
- FASTQC diagram of read quality for each sample
- Assembled, merged paired-end reads ready for Part 2 OTU clustering or ASV inference
- BDS checkpoint files enabling partial re-execution of later pipeline parts

## How to apply

Place the .fastq.gz files in a subdirectory named 'mydata' inside the analysis directory and configure the parameters.tsv file for the 16S, 18S, 12S, ITS, or COI marker gene. Start with FastQC to produce a per-sample read-quality summary, then run Trimmomatic for adapter, primer, and quality trimming — parameter settings for this step must be tuned per dataset, as the authors tested many settings especially for read quality trimming. For ITS data, add Cutadapt to remove primers before continuing. Run BayesHammer (from SPAdes) to correct base errors, then PANDAseq to assemble the overlapping paired-end reads, and finally the obiuniq program of OBITools to group all identical sequences per sample while keeping track of their abundances. Invoke the VSEARCH package for chimera removal, noting that when ASV inference with Swarm v2 is used downstream, chimera removal is deferred until after ASV inference. Run inside the containerized pipeline (Docker or Singularity) using a BDS checkpoint-based script so that changing Part 2 clustering/ASV algorithms does not force re-running Part 1.

## Related tools

- **FastQC** (Produces quality summary and FASTQC diagram of read quality for each sample before trimming) — https://github.com/hariszaf/pema
- **Trimmomatic** (Removes adapters, primers, and performs quality trimming of raw reads (parameterized per marker gene)) — https://github.com/hariszaf/pema
- **Cutadapt** (Invoked for primer removal in the ITS marker gene workflow) — https://github.com/hariszaf/pema
- **BayesHammer (SPAdes)** (Base error correction on trimmed reads before paired-end assembly) — https://github.com/hariszaf/pema
- **PANDAseq** (Assembles overlapping paired-end reads after error correction) — https://github.com/hariszaf/pema
- **obiuniq (OBITools)** (Groups all identical sequences per sample and keeps track of their abundances) — https://github.com/hariszaf/pema
- **VSEARCH** (Chimera removal after dereplication (deferred to post-ASV-inference when Swarm v2 is used)) — https://github.com/hariszaf/pema
- **PEMA** (Containerized pipeline wrapper (Docker/Singularity image, BDS script) orchestrating all Part 1 steps with checkpointing) — https://github.com/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- Each sample yields a FASTQC diagram and quality summary showing expected post-trimming quality-profile improvement.
- obiuniq output retains abundance counts for each unique sequence per sample (identical sequences grouped, not discarded).
- For ITS runs, primer removal via Cutadapt succeeded — verify no residual primer sequences before BayesHammer.
- When downstream clustering is Swarm v2, chimeras are removed after ASV inference, not in Part 1 — check the BDS log ordering.
- BDS checkpoint files are produced after Part 1 so that parameter changes in Part 2 trigger partial re-execution rather than a full rerun.

## Limitations

- Reads must be paired-end Illumina .fastq.gz; other formats or platforms are not supported by the pipeline.
- Optimal Trimmomatic parameters are dataset-dependent — many settings must be tested, especially for read quality trimming.
- Chimera removal must be deferred to after ASV inference when using Swarm v2; applying it earlier would break the ASV workflow.
- For COI, taxonomy assignment downstream requires a minimum of 20 GB of RAM, constraining where the full pipeline can run.
- PEMA will fail unless the analysis directory contains exactly the mandatory files (parameters.tsv and mydata subdirectory) with exact naming.

## Evidence

- [supplementary] Part 1 wiring of trimming/error-correction/assembly tools: [evidence span withheld — anchor status: not_found]
- [abstract] PEMA overall stage description: "PEMA performs read pre-processing, (molecular) operational taxonomic unit clustering, amplicon sequence variant inference, and taxonomy assignment for 16S and 18S ribosomal RNA, as well as ITS and"
- [methods] Step ordering and dereplication behavior: "PANDAseq [34] assembles the overlapping paired-end reads"
- [methods] Dereplication purpose: [evidence span withheld — anchor status: not_found]
- [methods] Chimera removal step: "The VSEARCH package [17] is then invoked for chimera removal"
- [methods] ITS-specific primer removal: [evidence span withheld — anchor status: not_found]
- [readme] Input format: [evidence span withheld — anchor status: not_found]
- [readme] Quality metrics reported: [evidence span withheld — anchor status: not_found]
- [methods] Parameter tuning burden: "Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference"
- [intro] Paired-end scope: "It is designed for paired-end sequencing studies and is implemented in the BDS [11] programming language"
- [abstract] Checkpoint behavior enabling partial re-execution: "Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution."
