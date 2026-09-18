---
name: swarm-v2-asv-inference-parameter-tuning
description: Use when paired-end amplicon reads (e.g., Illumina .fastq.gz for 18S/16S)
  must be resolved into ASVs rather than OTUs, and the choice of Swarm v2 strictness
  needs empirical benchmarking against a mock community.
when_to_use_negative:
- Input is already an ASV/OTU feature table with taxonomies rather than raw paired-end
  .fastq.gz reads
- 'The study requires VSEARCH OTU clustering instead of ASV inference (note: OTU/VSEARCH
  is not supported for COI and 12S in PEMA''s marker-gene matrix)'
- No reference set of expected taxa (e.g., a mock community) exists to compute precision/recall/F1
  for parameter selection
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: Swarm v2
  role: ASV inference engine invoked within PEMA under tuned parameters (d = 1, strictness
    0.5 / 0.3)
  repo: https://github.com/hariszaf/pema
- name: VSEARCH
  role: Chimera removal before clustering and the OTU-clustering alternative condition
    for comparison
- name: PEMA
  role: Containerized (Docker/Singularity) pipeline that orchestrates pre-processing,
    Swarm ASV inference, and taxonomy assignment
  repo: https://github.com/hariszaf/pema
- name: Docker
  role: Container runtime used to pull and run the PEMA image
  repo: https://hub.docker.com/r/hariszaf/pema
- name: CREST LCAClassifier
  role: Alignment-based taxonomy assignment of ASVs against the Silva database for
    16S/18S
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:2235a9c9af2b2c1f925bc9c55658edb40040ef24ac7132318c77f408b09da772
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# swarm-v2-asv-inference-parameter-tuning

## Summary

Tunes and runs Swarm v2 amplicon sequence variant (ASV) inference inside PEMA on 16S/18S (and other) metabarcoding reads, testing parameter combinations (d = 1 with strictness 0.5 vs 0.3) and validating against known mock-community composition with genus-level precision, recall, and F1.

## When to use

Use when paired-end amplicon reads (e.g., Illumina .fastq.gz for 18S/16S) must be resolved into ASVs rather than OTUs, and the choice of Swarm v2 strictness needs empirical benchmarking against a mock community.

## When NOT to use

- Input is already an ASV/OTU feature table with taxonomies rather than raw paired-end .fastq.gz reads
- The study requires VSEARCH OTU clustering instead of ASV inference (note: OTU/VSEARCH is not supported for COI and 12S in PEMA's marker-gene matrix)
- No reference set of expected taxa (e.g., a mock community) exists to compute precision/recall/F1 for parameter selection

## Inputs

- Illumina .fastq.gz paired-end metabarcoding reads per sample (e.g., Bradley et al. 18S rRNA mock community of 12 algal species from ENA)
- PEMA parameters.tsv file configured for ASV inference with Swarm v2 (d = 1; strictness tested at 0.5 and 0.3)
- Silva reference database (versions 128 or 132 bundled in PEMA) for CREST LCAClassifier taxonomy assignment
- List of known mock community members for precision/recall/F1 benchmarking

## Outputs

- ASV table with taxonomies and per-sample abundances
- FastQC quality diagram per sample and run statistics
- Genus-level precision, recall, and F1 scores per Swarm v2 parameter combination (F1 range observed: 74%–86%)

## How to apply

Run PEMA (Docker or Singularity container) so that Swarm v2 performs ASV inference after pre-processing: FastQC quality summary, Trimmomatic trimming, BayesHammer error correction, PANDAseq paired-end assembly, obiuniq dereplication, and VSEARCH chimera removal prior to clustering. Run Swarm v2 with d = 1 under at least two strictness settings — strictness = 0.5 and strictness = 0.3 — and, in parallel, cluster OTUs with VSEARCH as the alternative condition so results can be compared. Assign taxonomy with the CREST LCAClassifier against the Silva database, then compute genus-level precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) against the known mock community members. Optionally remove singletons (sequences with only 1 read) occurring after ASV inference. Select the parameter set that maximizes F1 while controlling false positives; in the Bradley et al. 18S algal mock community, strictness = 0.3 gave the best F1 of 0.82 with only 1 false positive, and strictness = 0.5 resolved 3 of 12 members to species level.

## Related tools

- **Swarm v2** (ASV inference engine invoked within PEMA under tuned parameters (d = 1, strictness 0.5 / 0.3)) — https://github.com/hariszaf/pema
- **VSEARCH** (Chimera removal before clustering and the OTU-clustering alternative condition for comparison)
- **PEMA** (Containerized (Docker/Singularity) pipeline that orchestrates pre-processing, Swarm ASV inference, and taxonomy assignment) — https://github.com/hariszaf/pema
- **Docker** (Container runtime used to pull and run the PEMA image) — https://hub.docker.com/r/hariszaf/pema
- **CREST LCAClassifier** (Alignment-based taxonomy assignment of ASVs against the Silva database for 16S/18S)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- Genus-level F1-scores for mock community recovery fall in the reported 74%–86% range
- Best F1 of 0.82 achieved at d = 1, strictness = 0.3 with only 1 false positive
- Under Swarm v2 (d = 1, strictness = 0.5), 3 of the 12 mock community members are identified to species level (Isochrysis galbana, Nannochloropsis oculata, Thalassiosira pseudonana)
- Chimeras were removed with VSEARCH before clustering/ASV inference; singletons (sequences with only 1 read) after ASV inference may be removed
- PEMA run completes with the checkpoint/parameterization of the pipeline and produces the ASV table with taxonomies and abundances per sample

## Limitations

- Many parameter settings had to be tested, especially for quality trimming and OTU clustering/ASV inference, so parameter tuning is data-dependent and compute-intensive
- COI taxonomy assignment requires a minimum of 20 GB of RAM, constraining where the full comparison workflow can run
- ASVs vs OTUs choice should be based on each study's objectives; neither approach is universally preferable
- PEMA will fail unless the analysis directory files and directories are named exactly as required (parameters.tsv, mydata, etc.)

## Evidence

- [intro] Swarm v2 allows ASV inference in all cases: "Swarm v2 [19] allows ASV inference in all cases"
- [methods] F1-score range for mock community recovery: "Overall the F1-scores ranged from 74% to 86%"
- [abstract] PEMA mock community evaluation quality: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [methods] Singleton removal after ASV inference: "any singletons, i.e., sequences with only 1 read, occurring after the (M)OTU clustering or the ASV inference may be removed"
- [readme] PEMA supports both OTU and ASV routes: [evidence span withheld — anchor status: not_found]
- [methods] Parameter testing burden: "Many different parameter settings were tested, especially for the steps of quality trimming of the reads and the OTU clustering/ASV inference"
