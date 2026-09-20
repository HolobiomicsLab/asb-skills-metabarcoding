---
name: amplicon-read-preprocessing-and-otu-clustering
description: Use when raw paired-end .fastq.gz amplicon reads need preprocessing (quality
  trimming, paired-end assembly, dereplication, chimera removal) and OTU clustering
  or ASV inference before taxonomic assignment and downstream diversity analysis.
when_to_use_negative:
- Input is already an OTU/ASV table or taxonomic profile rather than raw .fastq.gz
  reads — PEMA expects raw Illumina reads as input.
- The marker gene requires an OTU route PEMA does not support (e.g. VSEARCH OTUs for
  COI, or OTU clustering for 12S rRNA, where only Swarm ASV inference is available).
- A COI taxonomy assignment run on a machine with less than 20 GB RAM is planned,
  since PEMA requires a minimum of 20 GB of RAM for that step.
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: PEMA
  role: containerized pipeline orchestrating preprocessing, OTU clustering/ASV inference,
    and taxonomy assignment
  repo: https://github.com/hariszaf/pema
- name: PANDAseq
  role: assembles overlapping paired-end reads
- name: OBITools (obiuniq)
  role: groups identical sequences per sample keeping abundance counts
- name: VSEARCH
  role: (M)OTU clustering and chimera removal
- name: Swarm v2
  role: ASV inference
- name: CREST LCAClassifier
  role: alignment/phylogeny-based taxonomic assignment against Silva for 16S/18S
- name: phyloseq
  role: downstream ecological analysis of the resulting profiles
  repo: http://joey711.github.io/phyloseq/index.html
- name: Docker
  role: containerization of the pipeline (image hariszaf/pema)
  repo: https://hub.docker.com/r/hariszaf/pema
- name: BDS
  role: BigDataScript implementation providing task parallelism, roll-back checkpoints
    and partial pipeline execution
  repo: https://pcingola.github.io/BigDataScript/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:c7153db7477fa4a98fc37133144654f77b4fb2a58f3cda1df94f83d00df5cca8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - CREST LCAClassifier
  - Docker
  - LotuS
  - OBITools (obiuniq)
  - PANDAseq
  - PEMA
  - QIIME 2
  - Swarm v2
  - VSEARCH
  - mothur
  - phyloseq
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# amplicon-read-preprocessing-and-otu-clustering

## Summary

A method for turning raw paired-end Illumina .fastq.gz amplicon reads (16S/18S rRNA, ITS, COI) into quality-controlled, chimera- and singleton-filtered OTU/MOTU or ASV tables with taxonomy, as implemented in the containerized PEMA pipeline.

## When to use

Use when raw paired-end .fastq.gz amplicon reads need preprocessing (quality trimming, paired-end assembly, dereplication, chimera removal) and OTU clustering or ASV inference before taxonomic assignment and downstream diversity analysis.

## When NOT to use

- Input is already an OTU/ASV table or taxonomic profile rather than raw .fastq.gz reads — PEMA expects raw Illumina reads as input.
- The marker gene requires an OTU route PEMA does not support (e.g. VSEARCH OTUs for COI, or OTU clustering for 12S rRNA, where only Swarm ASV inference is available).
- A COI taxonomy assignment run on a machine with less than 20 GB RAM is planned, since PEMA requires a minimum of 20 GB of RAM for that step.

## Inputs

- paired-end Illumina .fastq.gz read files (per sample, placed in a mydata/ subdirectory)
- parameters.tsv parameter file (version-matched to the PEMA version, e.g. v.2.1.4)
- reference taxonomy databases: Silva 128/132, Unite, MIDORI (bundled or configured)

## Outputs

- OTU-table or ASV-table with taxonomies and per-sample abundances
- per-sample read-quality statistics and FASTQC quality diagrams
- optional phyloseq-based downstream analysis outputs (alpha/beta diversity, taxonomic composition)

## How to apply

Assemble overlapping paired-end reads with PANDAseq, group identical sequences per sample with obiuniq (OBITools) keeping abundance counts, then either cluster into (M)OTUs with VSEARCH or infer ASVs with Swarm v2; for 16S/18S/ITS both OTU and ASV routes are supported, while COI supports only Swarm ASVs and 12S only Swarm. Invoke VSEARCH again for chimera removal, and optionally drop singletons (sequences with only 1 read) after clustering/ASV inference. Assign taxonomy with the CREST LCAClassifier against Silva (16S/18S; versions 128 and 132 bundled) or Unite (ITS), or RDPClassifier against MIDORI (COI). Run the whole workflow via the PEMA Docker or Singularity image implemented in BigDataScript (BDS), which uses all available cores and supports roll-back checkpoints and partial re-execution, so parameter settings for quality trimming and OTU clustering/ASV inference can be retested without restarting the run. Parameters are set per run in the mandatory parameters.tsv file inside the analysis directory alongside the mydata folder of .fastq.gz inputs.

## Related tools

- **PEMA** (containerized pipeline orchestrating preprocessing, OTU clustering/ASV inference, and taxonomy assignment) — https://github.com/hariszaf/pema
- **PANDAseq** (assembles overlapping paired-end reads)
- **OBITools (obiuniq)** (groups identical sequences per sample keeping abundance counts)
- **VSEARCH** ((M)OTU clustering and chimera removal)
- **Swarm v2** (ASV inference)
- **CREST LCAClassifier** (alignment/phylogeny-based taxonomic assignment against Silva for 16S/18S)
- **phyloseq** (downstream ecological analysis of the resulting profiles) — http://joey711.github.io/phyloseq/index.html
- **Docker** (containerization of the pipeline (image hariszaf/pema)) — https://hub.docker.com/r/hariszaf/pema
- **BDS** (BigDataScript implementation providing task parallelism, roll-back checkpoints and partial pipeline execution) — https://pcingola.github.io/BigDataScript/

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- Analysis directory contains exactly the mandatory files/directories: parameters.tsv (version-matched, e.g. v.2.1.4), mydata/ with .fastq.gz files, and optionally phyloseq_in_PEMA.R and metadata.csv — PEMA fails otherwise.
- OTU/ASV table includes taxonomy strings and per-sample abundances; per-sample FASTQC quality diagrams and statistics are produced.
- Singleton filtering (sequences with 1 read) applied after clustering/ASV inference matches the parameters.tsv setting.
- For COI runs, taxonomy assignment completes only if at least 20 GB RAM was available; verify memory allocation if the step fails.
- Checkpoint/partial re-execution: changing parameters for quality trimming or OTU clustering/ASV inference should resume from the last checkpoint rather than re-running the full pipeline.

## Limitations

- Marker-gene support is asymmetric: VSEARCH OTU clustering is unavailable for COI and 12S rRNA; only Swarm ASV inference is offered for those markers.
- PEMA will fail unless files and directories are named exactly as specified (parameters.tsv, mydata, phyloseq_in_PEMA.R, metadata.csv).
- The parameters.tsv file must match the PEMA version being used (e.g. v.2.1.4 parameters with pema:v.2.1.4).
- For B. leachii no entry exists in the MIDORI_LONGEST_1.1 database, so some taxa cannot be assigned at COI.
- Third-party tool licenses are listed in Additional File 1 rather than in the article text, and software version numbers of components are not stated in the discussion.

## Evidence

- [abstract] PEMA performs read pre-processing, (molecular) operational taxonomic unit clustering, amplicon sequence variant inference, and taxonomy assignment for 16S and 18S ribosomal RNA: "PEMA performs read pre-processing, (molecular) operational taxonomic unit clustering, amplicon sequence variant inference, and taxonomy assignment"
- [intro] Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs: "Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs"
- [methods] PANDAseq [34] assembles the overlapping paired-end reads: "PANDAseq [34] assembles the overlapping paired-end reads"
- [methods] the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
- [methods] The VSEARCH package [17] is then invoked for chimera removal: "The VSEARCH package [17] is then invoked for chimera removal"
- [methods] any singletons, i.e., sequences with only 1 read, occurring after the (M)OTU clustering or the ASV inference may be removed: "any singletons, i.e., sequences with only 1 read, occurring after the (M)OTU clustering or the ASV inference may be removed"
- [abstract] Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution.: "Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution."
- [readme] PEMA will fail unless you name the aforementioned files and directories exactly as described above.: [evidence span withheld — anchor status: not_found]
