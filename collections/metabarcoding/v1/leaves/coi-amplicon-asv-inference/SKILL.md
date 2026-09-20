---
name: coi-amplicon-asv-inference
description: Use when raw paired-end COI .fastq.gz reads (e.g., from ENA) need pre-processing,
  ASV inference, and taxonomy assignment, and results must be validated on a mock
  community with known species composition.
when_to_use_negative:
- Input reads are 16S, 18S, ITS, or 12S amplicons rather than COI — the COI path supports
  only ASV inference (Swarm), not OTU clustering (VSEARCH).
- The analysis target is OTU clustering with VSEARCH or phylogeny-based taxonomy assignment,
  both of which PEMA supports only for 16S rRNA data.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: PEMA
  role: containerized pipeline (Docker/Singularity, BDS-implemented) orchestrating
    the full COI ASV workflow
  repo: https://github.com/hariszaf/pema
- name: FastQC
  role: per-sample read quality control and quality diagrams
- name: Trimmomatic
  role: quality trimming of raw reads
- name: BayesHammer
  role: read error correction during pre-processing
- name: PANDAseq
  role: assembles overlapping paired-end reads
- name: OBITools (obiuniq)
  role: groups identical sequences per sample, keeping track of abundances
- name: Swarm v2
  role: ASV inference for COI with cluster radius d = 10
- name: RDPClassifier
  role: assigns taxonomy of COI MOTUs/ASVs
- name: MIDORI
  role: quality-controlled metazoan mitochondrial COI reference database for taxonomy
    assignment
- name: VSEARCH
  role: invoked for chimera removal after clustering
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:5dea5c8e151a16f0edd93ec6842df5fd623fe2107e3a88c840f9ed75451836b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - FastQC
  - MIDORI
  - OBITools
  - OBITools (obiuniq)
  - PANDAseq
  - PEMA
  - RDPClassifier
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# coi-amplicon-asv-inference

## Summary

Run PEMA's COI metabarcoding workflow on raw paired-end Illumina reads, inferring ASVs with Swarm v2 and assigning taxonomy with RDPClassifier against MIDORI, then validating the resulting ASV table (e.g., against the Bista et al. COI mock community).

## When to use

Use when raw paired-end COI .fastq.gz reads (e.g., from ENA) need pre-processing, ASV inference, and taxonomy assignment, and results must be validated on a mock community with known species composition.

## When NOT to use

- Input reads are 16S, 18S, ITS, or 12S amplicons rather than COI — the COI path supports only ASV inference (Swarm), not OTU clustering (VSEARCH).
- The analysis target is OTU clustering with VSEARCH or phylogeny-based taxonomy assignment, both of which PEMA supports only for 16S rRNA data.

## Inputs

- raw paired-end COI .fastq.gz read files (e.g., Bista et al. COI mock community reads from ENA)
- parameters.tsv file matching the PEMA version being used
- MIDORI reference database (e.g., MIDORI_LONGEST_1.1) bundled with PEMA

## Outputs

- ASV/MOTU table with taxonomic assignments and per-sample abundances
- per-sample read-quality statistics and FastQC diagrams
- validated mock-community recovery (12 of 14 expected species identified)

## How to apply

Retrieve the COI .fastq read files (e.g., Bista et al. mock community from ENA) and place them in a 'mydata' subdirectory of an analysis directory containing a version-matched parameters.tsv. Run PEMA's quality control and pre-processing chain (FastQC, Trimmomatic, BayesHammer, PANDAseq for pairing overlapping reads, obiuniq for grouping identical sequences and tracking abundances). Infer ASVs with Swarm v2 using cluster radius d = 10, letting PEMA handle chimera removal after clustering since Swarm v2 is not affected by chimeras. Assign taxonomy with RDPClassifier against the MIDORI reference database, noting that COI taxonomy assignment requires at least 20 GB of RAM. Compile the MOTU/ASV table with abundances and taxonomies per sample and validate against mock community expectations (12 of the 14 Bista mock community species should be recovered).

## Related tools

- **PEMA** (containerized pipeline (Docker/Singularity, BDS-implemented) orchestrating the full COI ASV workflow) — https://github.com/hariszaf/pema
- **FastQC** (per-sample read quality control and quality diagrams)
- **Trimmomatic** (quality trimming of raw reads)
- **BayesHammer** (read error correction during pre-processing)
- **PANDAseq** (assembles overlapping paired-end reads)
- **OBITools (obiuniq)** (groups identical sequences per sample, keeping track of abundances)
- **Swarm v2** (ASV inference for COI with cluster radius d = 10)
- **RDPClassifier** (assigns taxonomy of COI MOTUs/ASVs)
- **MIDORI** (quality-controlled metazoan mitochondrial COI reference database for taxonomy assignment)
- **VSEARCH** (invoked for chimera removal after clustering)

## Examples

```
singularity run -B /path/to/analysis_directory/:/mnt/analysis /path/to/pema_v.2.1.4.sif
```

## Evaluation signals

- Mock community validation: the taxonomy-assigned table identifies 12 of the 14 Bista et al. mock community species.
- The ASV table reports abundances and taxonomies per sample with no unassigned-taxonomy artifacts from missing MIDORI entries (e.g., B. leachii is absent from MIDORI_LONGEST_1.1).
- PEMA's overall F1-scores for mock community recovery at genus level fall in the 74%–86% range reported in the paper.
- Chimera removal is performed after clustering via VSEARCH, consistent with Swarm v2's insensitivity to chimeras.
- A per-sample FastQC quality diagram and statistics files are produced alongside the ASV table.

## Limitations

- COI taxonomy assignment requires a minimum of 20 GB of RAM, limiting which machines can run the full workflow.
- Some taxa have no entry in the MIDORI database (version MIDORI_LONGEST_1.1 lacks an entry for B. leachii), producing unassignable ASVs.
- For COI, PEMA supports only ASV inference with Swarm, not OTU clustering with VSEARCH.
- The parameters.tsv file must match the PEMA version exactly (e.g., v.2.1.4), or the run fails.

## Evidence

- [intro] Swarm v2 [19] allows ASV inference in all cases: "Swarm v2 [19] allows ASV inference in all cases"
- [methods] PEMA uses the RDPClassifier and the MIDORI reference database to assign taxonomy of the MOTUs: "PEMA uses the RDPClassifier [25] and the MIDORI reference database [26] to assign taxonomy of the MOTUs"
- [methods] the VSEARCH package is then invoked for chimera removal: "The VSEARCH package [17] is then invoked for chimera removal"
- [methods] the obiuniq program of OBITools groups all the identical sequences in every sample, keeping track of their abundances: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
- [methods] Overall the F1-scores ranged from 74% to 86%: "Overall the F1-scores ranged from 74% to 86%"
- [readme] For COI, a minimum of 20 GB of RAM for the taxonomy assignment step is required.: [evidence span withheld — anchor status: not_found]
