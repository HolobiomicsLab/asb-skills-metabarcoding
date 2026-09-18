---
name: 16s-rrna-otu-clustering-parameter-tuning
description: Use when benchmarking a 16S rRNA metabarcoding pipeline on mock-community
  FASTQ reads and you must choose or justify OTU/ASV clustering parameters (e.g. Swarm
  d distance, VSEARCH vs ASV inference) using precision/recall/F1 at genus level.
when_to_use_negative:
- The input is already an OTU/ASV table or feature table rather than raw paired-end
  .fastq.gz reads
- The marker gene is ITS, COI, 12S or 18S — the mock-community benchmark conditions
  here (Swarm d=3/d=30, CREST+Silva genus-level scoring) are specific to 16S rRNA
  bacterial data
- No ground-truth mock community or reference taxa list exists, making precision/recall/F1
  computation impossible
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: PEMA
  role: Containerized pipeline whose parameters.tsv is edited to run alternative OTU
    clustering / ASV inference conditions with checkpoint-based partial re-execution
  repo: https://github.com/hariszaf/pema
- name: Swarm v2
  role: ASV inference engine tuned with d=3 vs d=30 and strictness=0.6 across benchmark
    conditions
- name: VSEARCH
  role: OTU clustering condition C and chimera removal after obiuniq dereplication
- name: CREST LCAClassifier
  role: Alignment/phylogeny-based genus-level taxonomy assignment against Silva 128/132
    for each condition
- name: PANDAseq
  role: Assembles overlapping paired-end reads before dereplication
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:874018ded7727d307b759ffa347afd39110fed41fd5d5cdb0f24c592e44e7bcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BayesHammer
  - CREST LCAClassifier
  - FastQC
  - OBITools
  - PANDAseq
  - PEMA
  - Swarm v2
  - Trimmomatic
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# 16S rRNA OTU clustering parameter tuning

## Summary

Benchmarking OTU-clustering and ASV-inference parameter settings (VSEARCH OTUs vs Swarm v2 ASVs, e.g. d=3 vs d=30) on 16S rRNA mock-community data, scoring each condition by genus-level precision, recall and F1 against the Gohl et al. 20-species bacterial mock community.

## When to use

Use when benchmarking a 16S rRNA metabarcoding pipeline on mock-community FASTQ reads and you must choose or justify OTU/ASV clustering parameters (e.g. Swarm d distance, VSEARCH vs ASV inference) using precision/recall/F1 at genus level.

## When NOT to use

- The input is already an OTU/ASV table or feature table rather than raw paired-end .fastq.gz reads
- The marker gene is ITS, COI, 12S or 18S — the mock-community benchmark conditions here (Swarm d=3/d=30, CREST+Silva genus-level scoring) are specific to 16S rRNA bacterial data
- No ground-truth mock community or reference taxa list exists, making precision/recall/F1 computation impossible

## Inputs

- Paired-end 16S rRNA .fastq.gz reads (e.g. Gohl et al. 20-species bacterial mock community from ENA)
- Silva 128 and/or Silva 132 reference database
- PEMA parameters.tsv configured per clustering condition

## Outputs

- Per-condition OTU/ASV tables with taxonomic assignments at genus and species level
- Precision, recall and F1 scores per clustering condition
- Counts of mock-community taxa recovered to genus and species level per run

## How to apply

Run the pre-processing chain first: FastQC quality summary, Trimmomatic adapter/primer trimming and quality filtering, BayesHammer (SPAdes) error correction, PANDAseq paired-end assembly, obiuniq dereplication, and VSEARCH chimera removal. Then run the same data under alternative clustering conditions: condition A with Swarm v2 (d=3, strictness=0.6) without singleton removal for ASV inference (perform chimera removal after ASV inference since Swarm v2 clouds are not affected by chimeras); condition B with Swarm v2 (d=30, strictness=0.6) to minimize false-positive genus-level assignments; condition C with VSEARCH OTU clustering. Assign taxonomy for each condition with the CREST LCAClassifier against Silva (versions 128/132) at genus level. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 = 2*(precision*recall)/(precision+recall) per condition, and count taxa resolved to genus and species level. PEMA's checkpoint support means alternative conditions can be tested without full re-execution.

## Related tools

- **PEMA** (Containerized pipeline whose parameters.tsv is edited to run alternative OTU clustering / ASV inference conditions with checkpoint-based partial re-execution) — https://github.com/hariszaf/pema
- **Swarm v2** (ASV inference engine tuned with d=3 vs d=30 and strictness=0.6 across benchmark conditions)
- **VSEARCH** (OTU clustering condition C and chimera removal after obiuniq dereplication)
- **CREST LCAClassifier** (Alignment/phylogeny-based genus-level taxonomy assignment against Silva 128/132 for each condition)
- **PANDAseq** (Assembles overlapping paired-end reads before dereplication)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- Swarm v2 (d=3) run recovers 18 of 20 mock-community taxa to genus level
- Per-condition precision and recall scores are ~80% or above
- F1-scores for genus-level mock-community recovery fall in the reported 74%–86% range
- VSEARCH OTU clustering condition shows high precision (>0.79) but recall of 0.65–0.68
- Swarm v2 (d=30) condition reduces false-positive genus-level assignments relative to d=3

## Limitations

- Swarm v2 output clouds are not affected by chimeras, so chimera removal must be performed after ASV inference rather than before, unlike the VSEARCH OTU path
- VSEARCH OTU clustering trades recall for precision (recall 0.65–0.68 despite precision >0.79)
- Choice of ASVs vs OTUs should be based on each study's objective(s); no single clustering setting dominates all metrics
- For COI, at least 20 GB RAM is required for the taxonomy assignment step, so benchmarking on a PC may not be feasible for other marker genes

## Evidence

- [methods] F1 range: "Overall the F1-scores ranged from 74% to 86%"
- [abstract] mock community evaluation: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [intro] clustering algorithms: "Two clustering algorithms, VSEARCH [17] and CROP [18], are used for the clustering of reads in (M)OTUs"
- [methods] chimera removal step: "The VSEARCH package [17] is then invoked for chimera removal"
- [intro] ASV inference: "Swarm v2 [19] allows ASV inference in all cases"
- [abstract] parameter exploration via checkpoints: "Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution."
- [readme] OTU vs ASV support: [evidence span withheld — anchor status: not_found]
