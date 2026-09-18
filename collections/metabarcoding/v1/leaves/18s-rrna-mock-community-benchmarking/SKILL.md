---
name: 18s-rrna-mock-community-benchmarking
description: Use when reference for validating that a metabarcoding pipeline (e.g.,
  PEMA) reproduces published mock-community quality benchmarks when running 18S rRNA
  amplicon data with known species composition.
when_to_use_negative:
- The community composition is unknown (no ground-truth taxa list), so precision/recall
  against known members cannot be computed.
- The input data target a different marker gene (e.g., COI, which in PEMA supports
  only ASV inference via Swarm and requires ≥20 GB RAM for taxonomy assignment).
- Input is already an ASV/OTU table rather than raw .fastq.gz reads, since the skill
  benchmarks the full read-to-taxonomy pipeline.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0622
tools:
- name: PEMA
  role: containerized pipeline under evaluation; runs read pre-processing, clustering,
    ASV inference, and taxonomy assignment
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference (d = 1 with strictness 0.5 and 0.3 tested)
- name: VSEARCH
  role: chimera removal prior to clustering and alternative OTU clustering condition
- name: PANDAseq
  role: assembles overlapping paired-end reads
- name: obiuniq (OBITools)
  role: groups identical sequences per sample, keeping track of abundances
- name: CREST LCAClassifier
  role: alignment-based taxonomic assignment for 16S/18S rRNA data
- name: Silva database
  role: reference database for 18S/16S taxonomy assignment (versions 128 and 132 included)
- name: Docker
  role: containerization technology for running PEMA (docker pull hariszaf/pema)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:831694eab6c53016c522f00b8ec445afb827624137db40031cdd44d81cfbc0c2
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
  - Silva database
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  - obiuniq (OBITools)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# 18S rRNA mock community benchmarking

## Summary

Benchmark an eDNA metabarcoding pipeline's 18S rRNA analysis against a mock community (e.g., the Bradley et al. 12-algal-species dataset) by comparing predicted taxa to known members and computing genus-level precision, recall, and F1-scores.

## When to use

Reference for validating that a metabarcoding pipeline (e.g., PEMA) reproduces published mock-community quality benchmarks when running 18S rRNA amplicon data with known species composition.

## When NOT to use

- The community composition is unknown (no ground-truth taxa list), so precision/recall against known members cannot be computed.
- The input data target a different marker gene (e.g., COI, which in PEMA supports only ASV inference via Swarm and requires ≥20 GB RAM for taxonomy assignment).
- Input is already an ASV/OTU table rather than raw .fastq.gz reads, since the skill benchmarks the full read-to-taxonomy pipeline.

## Inputs

- 18S rRNA mock community paired-end .fastq files (Bradley et al. 12-algal-species dataset, from ENA)
- parameters.tsv configured for the 18S rRNA marker gene (Swarm d = 1, strictness 0.5 and 0.3; VSEARCH clustering)
- List of the 12 known mock community member taxa for truth comparison

## Outputs

- ASV table (Swarm v2) and OTU table (VSEARCH) with taxonomies and per-sample abundances
- Genus-level precision, recall, and F1-score per Swarm/VSEARCH parameter combination
- Per-sample FastQC quality summaries and pipeline statistics

## How to apply

Download the Bradley et al. 18S mock community .fastq files (12 algal species) from ENA and run the pipeline in its container (Docker or Singularity), mounting an analysis directory containing the mandatory parameters.tsv and a mydata/ folder with .fastq.gz files. Pre-process reads with FastQC, Trimmomatic, BayesHammer (SPAdes), PANDAseq assembly, and obiuniq (OBITools) dereplication; remove chimeras with VSEARCH before clustering. Infer ASVs with Swarm v2 under two parameter sets (d = 1, strictness = 0.5 and d = 1, strictness = 0.3) and, in parallel, cluster OTUs with VSEARCH as the alternative condition. Assign taxonomy with the CREST LCAClassifier against the Silva database, then compare predicted taxa to the 12 known members, computing precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) at genus level for each parameter combination.

## Related tools

- **PEMA** (containerized pipeline under evaluation; runs read pre-processing, clustering, ASV inference, and taxonomy assignment) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference (d = 1 with strictness 0.5 and 0.3 tested))
- **VSEARCH** (chimera removal prior to clustering and alternative OTU clustering condition)
- **PANDAseq** (assembles overlapping paired-end reads)
- **obiuniq (OBITools)** (groups identical sequences per sample, keeping track of abundances)
- **CREST LCAClassifier** (alignment-based taxonomic assignment for 16S/18S rRNA data)
- **Silva database** (reference database for 18S/16S taxonomy assignment (versions 128 and 132 included))
- **Docker** (containerization technology for running PEMA (docker pull hariszaf/pema))

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- F1-score at genus level falls within the published 74%–86% range for the mock community.
- Best F1-score of 0.82 is reproduced at Swarm v2 settings d = 1, strictness = 0.3 with only 1 false positive.
- Under Swarm v2 (d = 1, strictness = 0.5), exactly 3 of the 12 mock community members are resolved to species level (Isochrysis galbana, Nannochloropsis oculata, Thalassiosira pseudonana).
- Species/genus/class-level resolution patterns are consistent with the reported benchmark.
- PEMA's output directory contains the OTU/ASV table with taxonomies, per-sample statistics, and FastQC quality diagrams as documented.

## Limitations

- PEMA version numbers and third-party tool licenses are not stated in the article text (deferred to Additional File 1); the parameters.tsv must match the PEMA version used (e.g., v.2.1.4).
- Results depend on parameter settings, especially quality trimming and OTU clustering/ASV inference steps; many settings had to be tested.
- No changelog or DOI for archived data was found in the article's discussion section; GigaDB archive is cited only by reference number.

## Evidence

- [methods] Swarm v2 d=1 strictness: [evidence span withheld — anchor status: not_found]
- [methods] F1 range: "Overall the F1-scores ranged from 74% to 86%"
- [abstract] mock community evaluation: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [abstract] pipeline scope: "PEMA performs read pre-processing, (molecular) operational taxonomic unit clustering, amplicon sequence variant inference, and taxonomy assignment"
- [readme] container-based run: [evidence span withheld — anchor status: not_found]
- [methods] filter step: "The VSEARCH package [17] is then invoked for chimera removal"
