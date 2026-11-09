---
name: fungal-taxonomy-assignment-against-unite
description: Use when the input is paired-end Illumina .fastq/.fastq.gz reads from
  fungal ITS marker-gene amplicon sequencing and a taxonomy-assigned OTU/ASV table
  with per-sample abundances is required, e.g. when reproducing the PEMA ITS mock-community
  evaluation.
when_to_use_negative:
- The marker gene is 16S/18S rRNA (use CREST LCAClassifier against Silva) or COI (use
  RDPClassifier against MIDORI) rather than fungal ITS
- Input is already an OTU/ASV table with taxonomies — no assignment step is needed
- 'Unpaired single-end reads: PEMA is designed for paired-end sequencing studies and
  accepts .fastq.gz from Illumina paired-end runs'
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: PEMA
  role: Containerized pipeline that orchestrates the ITS taxonomy-assignment step
    and assembles the annotated abundance table
  repo: https://github.com/hariszaf/pema
- name: CREST LCAClassifier
  role: Alignment-based classifier used to assign taxonomy to ITS OTUs/ASVs
- name: Unite database
  role: Fungal reference database paired with the ITS marker gene for taxonomic assignment
- name: Docker
  role: Containerization technology through which PEMA (and its bundled classifier/database)
    is run
  repo: https://hub.docker.com/r/hariszaf/pema
- name: Swarm v2
  role: ASV inference algorithm whose output units are taxonomically assigned (e.g.
    cluster radius d = 20 for ITS2)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:79573611f8bd12cf9596b3bd92e33bf0cb4005407d172e4d1084d8d065e044ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - CROP
  - Cutadapt
  - Docker
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - Unite database
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# Fungal taxonomy assignment against the Unite database (PEMA/CREST)

## Summary

Assigns taxonomy to ITS amplicon units (OTUs or Swarm-inferred ASVs) in PEMA using the alignment-based CREST LCAClassifier against the Unite fungal reference database, producing a taxonomically annotated table with per-sample abundances.

## When to use

Use when the input is paired-end Illumina .fastq/.fastq.gz reads from fungal ITS marker-gene amplicon sequencing and a taxonomy-assigned OTU/ASV table with per-sample abundances is required, e.g. when reproducing the PEMA ITS mock-community evaluation.

## When NOT to use

- The marker gene is 16S/18S rRNA (use CREST LCAClassifier against Silva) or COI (use RDPClassifier against MIDORI) rather than fungal ITS
- Input is already an OTU/ASV table with taxonomies — no assignment step is needed
- Unpaired single-end reads: PEMA is designed for paired-end sequencing studies and accepts .fastq.gz from Illumina paired-end runs

## Inputs

- Paired-end Illumina .fastq.gz reads of fungal ITS amplicons (e.g. ITS2 region)
- PEMA analysis directory with parameters.tsv (version-matched to the PEMA release) and mydata/ subdirectory
- Unite reference database (bundled with PEMA for the ITS marker gene)

## Outputs

- Taxonomically assigned OTU-table or ASV-table with taxa and per-sample abundances
- Per-sample read-quality statistics and FastQC diagram from PEMA

## How to apply

Run PEMA's ITS route inside its Docker or Singularity container: after pre-processing (Trimmomatic/Cutadapt, BayesHammer error correction, PANDAseq overlap assembly, obiuniq dereplication) and unit inference (VSEARCH OTU clustering or Swarm v2 ASV inference, e.g. Swarm d = 20 for ITS2), taxonomy is assigned with the alignment-based CREST LCAClassifier using the Unite database, which PEMA pairs with ITS exactly as Silva is paired with 16S/18S and MIDORI with COI. The classifier assigns taxa by alignment and reports the least common ancestor, so chimeras that survived inference remain the main risk; on the Bakker ITS mock community this setting resolved 19 fungal taxa but left Fusarium species unresolved beyond their family (Nectriaceae). The output is the taxonomically assigned OTU/ASV table with abundances per sample, which can be passed to PEMA's phyloseq/vegan downstream analysis. Judge correctness by checking that the table exists with taxonomies and abundances for every sample and that false-positive assignments stay within expected bounds for the chosen parameters (only 1 FP in the published ITS mock-community evaluation).

## Related tools

- **PEMA** (Containerized pipeline that orchestrates the ITS taxonomy-assignment step and assembles the annotated abundance table) — https://github.com/hariszaf/pema
- **CREST LCAClassifier** (Alignment-based classifier used to assign taxonomy to ITS OTUs/ASVs)
- **Unite database** (Fungal reference database paired with the ITS marker gene for taxonomic assignment)
- **Docker** (Containerization technology through which PEMA (and its bundled classifier/database) is run) — https://hub.docker.com/r/hariszaf/pema
- **Swarm v2** (ASV inference algorithm whose output units are taxonomically assigned (e.g. cluster radius d = 20 for ITS2))

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema && ./pema_latest.bds
```

## Evaluation signals

- The output ASV/OTU table exists with assigned taxonomies and per-sample abundances for every sample
- On the Bakker ITS mock community (19 fungal taxa), the genus Chytriomyces is detected in all 3 samples
- Fusarium species are not over-assigned: they remain unresolved beyond their family Nectriaceae
- False-positive taxonomic assignments do not exceed the single FP reported for this parameter setting
- The parameters.tsv used matches the PEMA version of the container (e.g. v.2.1.4 with pema_v.2.1.4.sif)

## Limitations

- Resolution is bounded by the Unite reference database: on the mock community, Fusarium could not be resolved beyond family level (Nectriaceae)
- Chimera removal is performed with VSEARCH after unit inference; chimeras not removed can contaminate taxonomic assignments
- Taxonomy assignment for COI requires a minimum of 20 GB RAM, so resource needs vary by marker gene
- Parameters must be re-tuned per dataset — many settings were tested especially for quality trimming and OTU clustering/ASV inference steps

## Evidence

- [intro] the Unite database [22] is used for the ITS gene: "the Unite database [22] is used for the ITS gene"
- [intro] Taxonomic assignment is performed in an alignment-based approach, making use of the CREST LCAClassifier [20]: "Taxonomic assignment is performed in an alignment-based approach, making use of the CREST LCAClassifier [20]"
- [methods] is used together with the Silva [21] and the Unite [22] database, respectively, to assign taxonomy to the OTUs: "is used together with the Silva [21] and the Unite [22] database, respectively, to assign taxonomy to the OTUs"
- [readme] PEMA accepts .fastq.gz files as returned by Illumina sequencing platforms: [evidence span withheld — anchor status: not_found]
- [other] Assign taxonomy to the inferred ASVs using the alignment-based CREST LCAClassifier against the Unite reference database: [evidence span withheld — anchor status: not_found]
