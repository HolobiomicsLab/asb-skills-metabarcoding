---
name: edna-metabarcoding-pipeline-execution
description: Use when raw paired-end Illumina .fastq.gz metabarcoding reads for 16S/18S/12S
  rRNA, ITS, or COI marker genes need processing from quality control through OTU
  clustering/ASV inference to taxonomy assignment in a single containerized run.
when_to_use_negative:
- Input is already an OTU/ASV feature table or processed amplicon data — PEMA starts
  from raw paired-end .fastq.gz reads
- Reads are not paired-end Illumina amplicon data for one of the supported marker
  genes (16S/18S/12S rRNA, ITS, COI)
- A targeted single-tool benchmark (e.g., only Swarm d=3 vs VSEARCH clustering) is
  needed without a full pipeline re-run — use PEMA's checkpoint/partial execution
  instead of a fresh full run
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0654
tools:
- name: PEMA
  role: Containerized metabarcoding pipeline orchestrating all steps from raw reads
    to taxonomy-assigned OTU/ASV tables
  repo: https://github.com/hariszaf/pema
- name: VSEARCH
  role: OTU clustering of reads into (M)OTUs and chimera removal
- name: Swarm v2
  role: ASV inference for all supported marker genes
- name: PANDAseq
  role: Assembles overlapping paired-end reads
- name: CREST LCAClassifier
  role: Alignment-based taxonomy assignment for 16S/18S rRNA against the Silva database
- name: RDPClassifier
  role: Taxonomy assignment of COI MOTUs against the MIDORI reference database
- name: OBITools (obiuniq)
  role: Groups identical sequences per sample, keeping track of abundances
- name: phyloseq
  role: R package facilitating downstream ecological and phylogenetic analysis
  repo: http://joey711.github.io/phyloseq/index.html
- name: Docker
  role: Containerization technology used to run PEMA across operating systems
  repo: https://hub.docker.com/r/hariszaf/pema
- name: Singularity
  role: HPC-centered containerization alternative for running PEMA
  repo: https://singularity-hub.org/collections/2295
merged_aliases:
- edna-metabarcoding-read-preprocessing
merged_alias_records:
- alias: edna-metabarcoding-read-preprocessing
  slug: edna-metabarcoding-read-preprocessing
  cosine_score: 0.8188
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:18e73024f1eabb80aabc6a0166c249c01204ba55e3161fb1b1846d3205f01af2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - Docker
  - FastQC
  - OBITools
  - OBITools (obiuniq)
  - PANDAseq
  - PEMA
  - RDPClassifier
  - Singularity
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  - phyloseq
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# edna-metabarcoding-pipeline-execution

## Summary

Execute the PEMA containerized pipeline end-to-end on Illumina .fastq.gz amplicon reads (16S/18S rRNA, ITS, COI, and since v.2.1.4 12S rRNA), producing an OTU- or ASV-table with taxonomies and per-sample abundances plus downstream phyloseq ecological analysis.

## When to use

Use when raw paired-end Illumina .fastq.gz metabarcoding reads for 16S/18S/12S rRNA, ITS, or COI marker genes need processing from quality control through OTU clustering/ASV inference to taxonomy assignment in a single containerized run.

## When NOT to use

- Input is already an OTU/ASV feature table or processed amplicon data — PEMA starts from raw paired-end .fastq.gz reads
- Reads are not paired-end Illumina amplicon data for one of the supported marker genes (16S/18S/12S rRNA, ITS, COI)
- A targeted single-tool benchmark (e.g., only Swarm d=3 vs VSEARCH clustering) is needed without a full pipeline re-run — use PEMA's checkpoint/partial execution instead of a fresh full run

## Inputs

- Paired-end Illumina .fastq.gz read files (per sample) placed in a mydata/ subdirectory
- parameters.tsv file matching the PEMA version being run
- metadata.csv (comma-separated, optional, for phyloseq downstream analysis)
- phyloseq_in_PEMA.R script (optional, for downstream ecological analysis)

## Outputs

- OTU- or ASV-table with taxonomies and abundances per sample
- Per-sample read quality statistics and FASTQC diagram
- MOTU/ASV abundance table with assigned taxonomy (Silva/Unite/MIDORI-based)
- phyloseq-based ecological analysis outputs (alpha/beta diversity, taxonomic composition, PERMANOVA) when requested

## How to apply

Pull the PEMA Docker (hariszaf/pema:v.2.1.4) or Singularity image, then build an analysis directory containing a version-matched parameters.tsv, a mydata/ subdirectory with the .fastq.gz reads, and optionally metadata.csv and phyloseq_in_PEMA.R for downstream analysis. Mount the analysis directory and run ./pema_latest.bds inside the container. PEMA pre-processes reads (FastQC, Trimmomatic quality trimming, BayesHammer error correction, PANDAseq paired-end assembly, obiuniq dereplication), removes chimeras with VSEARCH, clusters reads into MOTUs with VSEARCH/CROP or infers ASVs with Swarm v2, and assigns taxonomy — CREST LCAClassifier against Silva 128/132 for 16S/18S, Unite for ITS, RDPClassifier against MIDORI for COI. Choose OTU clustering vs ASV inference per marker gene support (e.g., COI supports ASVs/Swarm only) and study objectives, exploiting BDS checkpoint support to test alternative algorithms per step without full re-execution. Note that COI taxonomy assignment requires at least 20 GB of RAM.

## Related tools

- **PEMA** (Containerized metabarcoding pipeline orchestrating all steps from raw reads to taxonomy-assigned OTU/ASV tables) — https://github.com/hariszaf/pema
- **VSEARCH** (OTU clustering of reads into (M)OTUs and chimera removal)
- **Swarm v2** (ASV inference for all supported marker genes)
- **PANDAseq** (Assembles overlapping paired-end reads)
- **CREST LCAClassifier** (Alignment-based taxonomy assignment for 16S/18S rRNA against the Silva database)
- **RDPClassifier** (Taxonomy assignment of COI MOTUs against the MIDORI reference database)
- **OBITools (obiuniq)** (Groups identical sequences per sample, keeping track of abundances)
- **phyloseq** (R package facilitating downstream ecological and phylogenetic analysis) — http://joey711.github.io/phyloseq/index.html
- **Docker** (Containerization technology used to run PEMA across operating systems) — https://hub.docker.com/r/hariszaf/pema
- **Singularity** (HPC-centered containerization alternative for running PEMA) — https://singularity-hub.org/collections/2295

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- The analysis directory named exactly as PEMA expects (parameters.tsv, mydata/, optionally metadata.csv and phyloseq_in_PEMA.R) — PEMA fails otherwise
- The parameters.tsv version matches the PEMA image version being run (e.g., v.2.1.4 parameters file with pema:v.2.1.4)
- Output contains an OTU- or ASV-table with taxonomies and per-sample abundances plus per-sample FASTQC quality statistics
- For the 16S mock community benchmark (Gohl et al.), the Swarm v2 (d=3) run recovers 18 of 20 taxa to genus level with precision/recall ~80% or above and F1-scores in the 74%–86% range
- For the COI mock community benchmark (Bista et al.), 12 of the 14 mock community species are identified in the taxonomy-assigned table

## Limitations

- For COI, a minimum of 20 GB of RAM is required for the taxonomy assignment step
- PEMA is suggested to run on an HPC environment to exploit its full potential; a common PC can only suffice for analyses with a small number of samples
- HPC-centered Singularity vs Docker choice and third-party tool licenses are deferred to separate documentation (Additional File 1) not included in the main text
- Singletons occurring after (M)OTU clustering or ASV inference may need removal as a user decision, affecting downstream results

## Evidence

- [readme] PEMA supports the metabarcoding analysis of four marker genes, 16S rRNA (Bacteria), ITS (Fungi) as well as COI and 18S rRNA (metazoa). As input, PEMA accepts .fastq.gz files as returned by Illumina sequencing platforms.: [evidence span withheld — anchor status: not_found]
- [abstract] PEMA performs read pre-processing, (molecular) operational taxonomic unit clustering, amplicon sequence variant inference, and taxonomy assignment: "PEMA performs read pre-processing, (molecular) operational taxonomic unit clustering, amplicon sequence variant inference, and taxonomy assignment for 16S and 18S ribosomal RNA"
- [methods] PANDAseq [34] assembles the overlapping paired-end reads: "PANDAseq [34] assembles the overlapping paired-end reads"
- [methods] the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
- [methods] Overall the F1-scores ranged from 74% to 86%: "Overall the F1-scores ranged from 74% to 86%"
- [readme] For COI, a minimum of 20 GB of RAM for the taxonomy assignment step is required.: [evidence span withheld — anchor status: not_found]
