---
name: asv-inference-with-swarm
description: Use when running PEMA on quality-filtered, dereplicated Illumina .fastq.gz
  amplicon reads (16S/18S/ITS/12S/COI) and the study objective calls for ASV inference
  rather than VSEARCH OTU clustering.
when_to_use_negative:
- The study objective requires OTU clusters — PEMA's VSEARCH (or CROP) OTU clustering
  path should be used instead, especially since whether to use ASVs instead of OTUs
  should be based on each study's objectives
- Raw reads have not yet been through PEMA Part 1 (adapter/primer trimming, BayesHammer
  error correction, PANDAseq assembly, obiuniq dereplication) — Swarm should run on
  pre-processed reads
- Input data are not paired-end Illumina .fastq/.fastq.gz amplicon reads accepted
  by PEMA
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: PEMA
  role: Containerized pipeline (Docker/Singularity) that invokes Swarm v2 for ASV
    inference on 16S/18S/12S/ITS/COI data
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference algorithm executed by PEMA; cluster radius d is the key parameter
    (d=3 16S, d=20 ITS2, d=30 KAPA 16S)
- name: VSEARCH
  role: Invoked for chimera removal after Swarm v2 ASV inference
- name: CREST LCAClassifier
  role: Alignment-based taxonomy assignment of inferred ASVs against Silva (16S/18S)
    or Unite (ITS)
- name: OBITools
  role: obiuniq groups identical sequences per sample, keeping track of abundances,
    before ASV inference
merged_aliases:
- asv-inference-with-swarm-clustering
merged_alias_records:
- alias: asv-inference-with-swarm-clustering
  slug: asv-inference-with-swarm-clustering
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:8f29bf3c602f647abe1b4f53c2e7408ad94a9acffc71218ee0c890f077cc5733
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - CROP
  - Cutadapt
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# asv-inference-with-swarm

## Summary

Infer amplicon sequence variants (ASVs) from pre-processed eDNA metabarcoding reads using Swarm v2 as implemented in PEMA, with cluster radius tuned per marker gene (e.g. d=3 for 16S, d=20 for ITS2).

## When to use

Use when running PEMA on quality-filtered, dereplicated Illumina .fastq.gz amplicon reads (16S/18S/ITS/12S/COI) and the study objective calls for ASV inference rather than VSEARCH OTU clustering.

## When NOT to use

- The study objective requires OTU clusters — PEMA's VSEARCH (or CROP) OTU clustering path should be used instead, especially since whether to use ASVs instead of OTUs should be based on each study's objectives
- Raw reads have not yet been through PEMA Part 1 (adapter/primer trimming, BayesHammer error correction, PANDAseq assembly, obiuniq dereplication) — Swarm should run on pre-processed reads
- Input data are not paired-end Illumina .fastq/.fastq.gz amplicon reads accepted by PEMA

## Inputs

- Pre-processed, dereplicated amplicon sequences (post FastQC/Trimmomatic/BayesHammer/PANDAseq/obiuniq) per sample
- Raw Illumina .fastq.gz read pairs (e.g. Gohl et al. 16S mock community, Bakker ITS fungal mock community from ENA)
- PEMA parameters.tsv configured for ASV inference (Swarm v2) with marker-appropriate d value

## Outputs

- ASV table with per-sample abundances and assigned taxonomies (CREST LCAClassifier vs Silva 128/132 or Unite)
- Per-condition precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*precision*recall/(precision+recall) scores
- Counts of taxa identified to genus and species level per run

## How to apply

First complete PEMA Part 1 pre-processing: FastQC quality summaries, Trimmomatic (plus Cutadapt for ITS length variability) trimming, BayesHammer error correction, PANDAseq paired-end assembly, and obiuniq dereplication. Then run Swarm v2 in PEMA's parameters.tsv, selecting the cluster radius d for the marker: d=3 for 16S mock communities, d=30 on KAPA 16S samples to minimize false-positive genus assignments, and d=20 targeting the ITS2 region for fungal ITS data. Perform chimera removal AFTER ASV inference, because Swarm v2 output clouds are not affected by chimeras; optionally remove singletons (sequences with only 1 read) after clustering. Assign taxonomy to the inferred ASVs with the CREST LCAClassifier against Silva (16S/18S) or Unite (ITS), then compute precision, recall, and F1 to benchmark recovery of the mock community's expected taxa.

## Related tools

- **PEMA** (Containerized pipeline (Docker/Singularity) that invokes Swarm v2 for ASV inference on 16S/18S/12S/ITS/COI data) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference algorithm executed by PEMA; cluster radius d is the key parameter (d=3 16S, d=20 ITS2, d=30 KAPA 16S))
- **VSEARCH** (Invoked for chimera removal after Swarm v2 ASV inference)
- **CREST LCAClassifier** (Alignment-based taxonomy assignment of inferred ASVs against Silva (16S/18S) or Unite (ITS))
- **OBITools** (obiuniq groups identical sequences per sample, keeping track of abundances, before ASV inference)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds  # with parameters.tsv set to Swarm v2 (d=3 for 16S) for ASV inference
```

## Evaluation signals

- On the Gohl et al. 16S mock community (20 bacterial species), the Swarm v2 d=3 run recovers 18 of 20 taxa to genus level
- Per-condition precision/recall scores are ~80% or above, with F1-scores falling in the reported 74%-86% range
- On the Bakker ITS mock community (19 fungal taxa, d=20), genus Chytriomyces is detected in all 3 samples and false-positive assignments do not exceed the single FP reported
- The output ASV table exists with assigned taxonomies and per-sample abundances
- Chimera removal (VSEARCH) is executed after ASV inference, not before, for Swarm v2 runs

## Limitations

- Swarm v2 output clouds are not affected by chimeras, so chimeric sequences are not removed prior to ASV inference; VSEARCH chimera removal must run afterwards
- The cluster radius d must be tuned per marker and dataset: suboptimal d values increase false-positive genus-level assignments (hence d=30 on KAPA 16S samples)
- Overall F1-scores for mock community recovery at genus level (74%-86%) fall short of perfect recovery; e.g. VSEARCH OTU clustering reported high precision (>0.79) but recall of only 0.65-0.68

## Evidence

- [intro] Swarm v2 [19] allows ASV inference in all cases: "Swarm v2 [19] allows ASV inference in all cases"
- [intro] The use of ASVs instead of OTUs has been suggested; however, the choice of which approach to use should be based on each study's objective(s): "The use of ASVs instead of OTUs has been suggested [15]; however, the choice of which approach to use should be based on each study's objective(s)"
- [methods] Overall the F1-scores ranged from 74% to 86%: "Overall the F1-scores ranged from 74% to 86%"
- [methods] any singletons, i.e., sequences with only 1 read, occurring after the (M)OTU clustering or the ASV inference may be removed: "any singletons, i.e., sequences with only 1 read, occurring after the (M)OTU clustering or the ASV inference may be removed"
- [readme] PEMA supports both OTU clustering (VSEARCH) and ASV inference (Swarm).: [evidence span withheld — anchor status: not_found]
