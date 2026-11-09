---
name: mock-community-benchmark-evaluation
description: Use when a metabarcoding pipeline needs quantitative performance validation
  on a mock community with known taxonomic composition, e.g., comparing ASV inference
  (Swarm v2) vs OTU clustering (VSEARCH) precision/recall.
when_to_use_negative:
- The dataset is a real environmental community with unknown composition — precision/recall
  against ground truth cannot be computed
- Input is already an OTU/ASV table; this skill operates from raw .fastq.gz reads
  through the full pre-processing and inference chain
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_2814
tools:
- name: PEMA
  role: Containerized pipeline executing pre-processing, ASV inference / OTU clustering,
    and taxonomy assignment for the benchmark
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference (d=3 and d=30, strictness 0.6) in the benchmarked conditions
- name: VSEARCH
  role: OTU clustering condition and chimera removal step
- name: CREST LCAClassifier
  role: Alignment-based taxonomy assignment against Silva at genus level
- name: FastQC
  role: Read quality summary before trimming
- name: Trimmomatic
  role: Adapter/primer trimming and quality filtering
- name: PANDAseq
  role: Assembles overlapping paired-end reads
- name: OBITools (obiuniq)
  role: Groups identical sequences per sample keeping abundance counts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:b1cc61524c42ca6adbdb99e8d652b3d6431ecbd103b92134e0751763ccc5c867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - FastQC
  - OBITools
  - OBITools (obiuniq)
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_2814
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mock-community-benchmark-evaluation

## Summary

Benchmark an eDNA metabarcoding pipeline (e.g., PEMA) against a known-composition 16S rRNA mock community (Gohl et al., 20 bacterial species) by computing precision, recall, and F1 at genus level for alternative clustering/ASV-inference conditions.

## When to use

Use when a metabarcoding pipeline needs quantitative performance validation on a mock community with known taxonomic composition, e.g., comparing ASV inference (Swarm v2) vs OTU clustering (VSEARCH) precision/recall.

## When NOT to use

- The dataset is a real environmental community with unknown composition — precision/recall against ground truth cannot be computed
- Input is already an OTU/ASV table; this skill operates from raw .fastq.gz reads through the full pre-processing and inference chain

## Inputs

- Gohl et al. 16S rRNA mock community .fastq.gz paired-end reads (20 bacterial species) from ENA
- parameters.tsv file matched to the PEMA version being run
- Silva reference database (versions 128 and 132) for taxonomy assignment
- Expected mock community taxonomic composition (ground-truth taxa list)

## Outputs

- Per-condition ASV/OTU table with genus- and species-level taxonomic assignments and abundances
- Precision, recall, and F1-score per clustering/ASV-inference condition
- Count of taxa recovered to genus and species level per run

## How to apply

Run PEMA's Part 1 pre-processing on the Gohl et al. mock community .fastq.gz reads from ENA: FastQC quality summary, Trimmomatic adapter/primer trimming and quality filtering, BayesHammer (SPAdes) error correction, PANDAseq paired-end assembly, obiuniq dereplication, and VSEARCH chimera removal. Then run alternative conditions: Swarm v2 ASV inference with d=3 and strictness 0.6 (deferring chimera removal until after ASV inference since Swarm clouds are not affected by chimeras, without singleton removal), Swarm v2 with d=30 on KAPA samples to minimize false-positive genus assignments, and VSEARCH OTU clustering for comparison. Assign taxonomy for all conditions with the CREST LCAClassifier against Silva 128/132 at genus level. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) per condition, and record the number of taxa identified to genus and species level.

## Related tools

- **PEMA** (Containerized pipeline executing pre-processing, ASV inference / OTU clustering, and taxonomy assignment for the benchmark) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference (d=3 and d=30, strictness 0.6) in the benchmarked conditions)
- **VSEARCH** (OTU clustering condition and chimera removal step)
- **CREST LCAClassifier** (Alignment-based taxonomy assignment against Silva at genus level)
- **FastQC** (Read quality summary before trimming)
- **Trimmomatic** (Adapter/primer trimming and quality filtering)
- **PANDAseq** (Assembles overlapping paired-end reads)
- **OBITools (obiuniq)** (Groups identical sequences per sample keeping abundance counts)

## Examples

```
singularity run -B /<path_to_analysis_directory>/:/mnt/analysis /<path_to>/pema_v.2.1.4.sif && ./pema_latest.bds
```

## Evaluation signals

- Swarm v2 (d=3) run recovers 18 of 20 mock community taxa to genus level
- Per-condition precision/recall scores are ~80% or above
- F1-scores fall within the reported 74%–86% range
- VSEARCH OTU clustering condition shows precision >0.79 but recall of 0.65–0.68
- Taxonomy assignments consistent across Silva 128 and 132 versions at genus level

## Limitations

- Swarm v2 output clouds are not affected by chimeras, so chimera removal must be performed after ASV inference rather than before
- F1 and recall are sensitive to parameter settings, especially quality trimming and OTU clustering/ASV inference parameters, which required extensive testing
- Results depend on the Silva database version used (128 vs 132); taxonomy assignment quality varies with reference database coverage

## Evidence

- [abstract] PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality.: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [methods] F1 range: "Overall the F1-scores ranged from 74% to 86%"
- [intro] Swarm ASV inference: "Swarm v2 [19] allows ASV inference in all cases"
- [intro] CREST taxonomy: "Taxonomic assignment is performed in an alignment-based approach, making use of the CREST LCAClassifier [20] and the Silva database [21]"
- [methods] Silva versions: "Two versions of Silva are included in PEMA: 128 (29 September 2016) and 132 (13 December 2017)"
- [methods] VSEARCH chimera removal: "The VSEARCH package [17] is then invoked for chimera removal"
- [methods] obiuniq dereplication: "the “obiuniq” program of OBITools [35] groups all the identical sequences in every sample, keeping track of their abundances"
