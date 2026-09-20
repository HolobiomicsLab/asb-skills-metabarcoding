---
name: precision-recall-f1-benchmark-scoring
description: Use when a pipeline's taxonomy assignments must be validated against
  a mock community with known member taxa, and comparable quality metrics (precision,
  recall, F1) to prior benchmarks are needed.
when_to_use_negative:
- The dataset is a real environmental community with unknown composition — precision/recall
  against a truth set is undefined; use ecological metrics (alpha/beta diversity)
  instead.
- The pipeline output is already a validated feature/OTU table with published benchmark
  scores — re-deriving metrics would duplicate existing evaluation.
- Marker genes other than 16S/18S rRNA processed under conditions where taxonomy assignment
  (e.g., MIDORI for COI) lacks reference entries for known members, making TP/FP classification
  unreliable.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PEMA
  role: Containerized pipeline whose mock-community taxonomic output (ASV/OTU tables
    with taxonomy) is scored
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference under d = 1 with strictness 0.5 and 0.3, generating the variants
    scored against the mock community
- name: VSEARCH
  role: OTU clustering alternative condition and chimera removal before scoring
- name: CREST LCAClassifier
  role: Genus-level taxonomy assignment of ASVs/OTUs against the Silva database, providing
    the predicted taxa for TP/FP classification
- name: Docker
  role: Containerization used to run PEMA on the mock community reads reproducibly
  repo: https://hub.docker.com/r/hariszaf/pema
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:75cbf6b406060490ac08b6f05ec06f2991775cac0d41e06b8856f90a77e36aeb
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
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Precision-Recall-F1 benchmark scoring

## Summary

Benchmark scoring of a metabarcoding pipeline's taxonomic output against a mock community with known composition, computing genus-level precision, recall, and F1-score from true/false positives and negatives. Used to quantify how faithfully PEMA's ASV/OTU tables recover the expected taxa of the Bradley et al. 18S algal mock community.

## When to use

Use when a pipeline's taxonomy assignments must be validated against a mock community with known member taxa, and comparable quality metrics (precision, recall, F1) to prior benchmarks are needed.

## When NOT to use

- The dataset is a real environmental community with unknown composition — precision/recall against a truth set is undefined; use ecological metrics (alpha/beta diversity) instead.
- The pipeline output is already a validated feature/OTU table with published benchmark scores — re-deriving metrics would duplicate existing evaluation.
- Marker genes other than 16S/18S rRNA processed under conditions where taxonomy assignment (e.g., MIDORI for COI) lacks reference entries for known members, making TP/FP classification unreliable.

## Inputs

- Illumina .fastq.gz paired-end reads of a mock community (Bradley et al. 18S rRNA dataset, 12 algal species)
- ASV or OTU table with genus-level taxonomic assignments (from Swarm v2 or VSEARCH + CREST LCAClassifier/Silva)
- Ground-truth list of the known mock community member taxa (12 algal species/genera)

## Outputs

- Genus-level precision, recall, and F1-score per parameter combination (e.g., Swarm strictness 0.5 vs 0.3; OTU vs ASV condition)
- Count of true positives, false positives, and false negatives against the 12 known community members
- Species-level resolution summary (e.g., number of members identified to species level)

## How to apply

Run the pipeline (e.g., PEMA via Docker/Singularity) on the mock community .fastq.gz reads, generating an ASV table (Swarm v2 with d = 1 under two strictness settings, 0.5 and 0.3) and/or an OTU table (VSEARCH) with CREST LCAClassifier taxonomy assignment against the Silva database. Match the predicted taxa to the known mock community members at the genus level, defining TP as correctly predicted member genera, FP as predicted non-member genera, and FN as missed member genera. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) for each parameter combination. Compare resulting F1-scores against the published benchmark range (74%–86%) to judge comparable quality; the reference run achieved a best F1-score of 0.82 at d = 1, strictness = 0.3 with only 1 false positive, and identified 3 of 12 members to species level (Isochrysis galbana, Nannochloropsis oculata, Thalassiosira pseudonana) at strictness = 0.5.

## Related tools

- **PEMA** (Containerized pipeline whose mock-community taxonomic output (ASV/OTU tables with taxonomy) is scored) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference under d = 1 with strictness 0.5 and 0.3, generating the variants scored against the mock community)
- **VSEARCH** (OTU clustering alternative condition and chimera removal before scoring)
- **CREST LCAClassifier** (Genus-level taxonomy assignment of ASVs/OTUs against the Silva database, providing the predicted taxa for TP/FP classification)
- **Docker** (Containerization used to run PEMA on the mock community reads reproducibly) — https://hub.docker.com/r/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- F1-scores fall within the published benchmark range of 74%–86% for genus-level mock community recovery
- Best F1-score of 0.82 is reproduced at Swarm d = 1, strictness = 0.3, with exactly 1 false positive
- 3 of the 12 mock community members are resolved to species level (Isochrysis galbana, Nannochloropsis oculata, Thalassiosira pseudonana) under Swarm v2 (d = 1, strictness = 0.5)
- Precision and recall decompose consistently: precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+receipt+... ) holds arithmetically for the reported counts
- OTU (VSEARCH) vs ASV (Swarm) conditions are both scored so parameter-set comparisons are like-for-like at the genus level

## Limitations

- Evaluation is only as good as the reference database: MIDORI_LONGEST_1.1, for example, lacks an entry for B. leachii, showing database gaps can cause false negatives unrelated to pipeline performance.
- Reported metrics are specific to genus-level resolution on a 12-species 18S algal mock community; they do not transfer directly to other markers (16S, ITS, COI) or communities.
- Software version numbers for PEMA components were not stated in the discussion, so exact reproducibility may depend on matching tool versions separately.

## Evidence

- [methods] F1 range benchmark: "Overall the F1-scores ranged from 74% to 86%"
- [abstract] comparable quality claim: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [methods] precision/recall/F1 definitions: [evidence span withheld — anchor status: not_found]
- [readme] pipeline output scored: [evidence span withheld — anchor status: not_found]
