---
name: mock-community-species-recovery-evaluation
description: Use when benchmarking a COI metabarcoding pipeline against a mock community
  of known composition (e.g. Bista et al.) to measure species recovery and genus-level
  F1-scores.
when_to_use_negative:
- Analysis targets 16S/18S rRNA or ITS markers where OTU clustering via VSEARCH (not
  COI ASV inference via Swarm) is the appropriate PEMA route
- Input data are already a taxonomy-assigned ASV/MOTU table rather than raw .fastq.gz
  reads
- The sample is a real environmental community rather than a mock community with a
  known species inventory, so no ground-truth recovery rate can be computed
edam_operation: http://edamontology.org/operation_3222
tools:
- name: PEMA
  role: containerized pipeline orchestrating QC, ASV inference, and taxonomy assignment
    for the COI mock community evaluation
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference algorithm invoked with cluster radius d = 10; not affected by
    chimeras
- name: RDPClassifier
  role: assigns taxonomy of the COI MOTUs/ASVs
- name: MIDORI
  role: quality-controlled metazoan mitochondrial COI reference database used for
    taxonomic assignment
- name: PANDAseq
  role: assembles the overlapping paired-end COI reads during pre-processing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:2a4af6ad674b973df3efd4c6f6e68db0e5db99d448a3a3af95d1b66819eaff2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - FastQC
  - MIDORI
  - OBITools
  - PANDAseq
  - PEMA
  - RDPClassifier
  - Swarm v2
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# mock-community-species-recovery-evaluation

## Summary

Evaluates an eDNA metabarcoding pipeline (PEMA) on a COI mock community (Bista et al. dataset) by running the full read-preprocessing, ASV-inference (Swarm v2, d=10), and taxonomy-assignment (RDPClassifier + MIDORI) workflow and quantifying how many of the known mock community species are recovered.

## When to use

Use when benchmarking a COI metabarcoding pipeline against a mock community of known composition (e.g. Bista et al.) to measure species recovery and genus-level F1-scores.

## When NOT to use

- Analysis targets 16S/18S rRNA or ITS markers where OTU clustering via VSEARCH (not COI ASV inference via Swarm) is the appropriate PEMA route
- Input data are already a taxonomy-assigned ASV/MOTU table rather than raw .fastq.gz reads
- The sample is a real environmental community rather than a mock community with a known species inventory, so no ground-truth recovery rate can be computed

## Inputs

- Bista et al. COI mock community paired-end .fastq.gz reads (from ENA/EBI)
- PEMA parameters.tsv configuration file (version-matched to the PEMA release)
- MIDORI reference database (MIDORI_LONGEST_1.1) for COI taxonomy assignment

## Outputs

- ASV/MOTU table with abundances per sample and RDPClassifier/MIDORI-assigned taxonomies
- Mock community recovery assessment: list of identified species (12 of 14 expected)
- Genus-level F1-scores (expected range 74–86%)

## How to apply

Retrieve the Bista et al. COI mock community paired-end .fastq.gz reads from ENA and place them in PEMA's mydata directory. Run PEMA's quality control and pre-processing chain (FastQC, Trimmomatic, BayesHammer, PANDAseq, obiuniq), then infer ASVs with Swarm v2 using cluster radius d = 10, letting PEMA handle chimera removal after clustering since Swarm v2 is not affected by chimeras. Assign taxonomy with RDPClassifier against the MIDORI reference database (MIDORI_LONGEST_1.1 for the COI marker), noting that for COI a minimum of 20 GB of RAM is required for the taxonomy assignment step. Compile the MOTU/ASV table with abundances and taxonomies per sample, then validate by verifying that 12 of the 14 mock community species are identified and computing F1-scores at genus level (PEMA's mock community F1-scores ranged from 74% to 86%).

## Related tools

- **PEMA** (containerized pipeline orchestrating QC, ASV inference, and taxonomy assignment for the COI mock community evaluation) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference algorithm invoked with cluster radius d = 10; not affected by chimeras)
- **RDPClassifier** (assigns taxonomy of the COI MOTUs/ASVs)
- **MIDORI** (quality-controlled metazoan mitochondrial COI reference database used for taxonomic assignment)
- **PANDAseq** (assembles the overlapping paired-end COI reads during pre-processing)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema && ./pema_latest.bds
```

## Evaluation signals

- Species-recovery invariant: 12 of the 14 mock community species appear in the taxonomy-assigned table
- Genus-level F1-scores fall in the reported 74–86% range
- Swarm cluster radius is set to d = 10 and chimera removal occurs after clustering
- Taxonomy assignments are made against MIDORI_LONGEST_1.1; known gaps (e.g. B. leachii missing from MIDORI) are documented rather than treated as pipeline failures
- PEMA completes the COI taxonomy assignment step, which requires a minimum of 20 GB of RAM

## Limitations

- B. leachii has no entry in the MIDORI database version MIDORI_LONGEST_1.1, so some mock community taxa cannot be taxonomically assigned
- For COI, a minimum of 20 GB of RAM is required for the taxonomy assignment step
- PEMA supports only ASV inference (Swarm) for COI; OTU clustering via VSEARCH is not available for this marker gene
- Results are 'of comparable quality' rather than exactly reproducing ground truth; deviations up to the reported F1-score range are expected

## Evidence

- [abstract] PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality.: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [methods] F1-score range for mock community recovery: "Overall the F1-scores ranged from 74% to 86%"
- [methods] MIDORI coverage gap limiting COI taxonomy assignment: "For B. leachii no entry exists in the MIDORI database, version MIDORI_LONGEST_1.1."
- [methods] MIDORI is the quality-controlled COI reference database built from GenBank: "The MIDORI database contains quality-controlled metazoan mitochondrial gene sequences from GenBank"
- [readme] COI taxonomy assignment memory requirement: [evidence span withheld — anchor status: not_found]
- [readme] COI supports ASV inference via Swarm but not OTU clustering via VSEARCH: [evidence span withheld — anchor status: not_found]
