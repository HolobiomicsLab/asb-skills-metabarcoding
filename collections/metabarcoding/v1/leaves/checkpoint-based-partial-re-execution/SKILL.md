---
name: checkpoint-based-partial-re-execution
description: Use when building or modifying a multi-step pipeline where users may
  swap tools or tweak parameters (e.g., quality trimming, OTU clustering/ASV inference)
  and must avoid re-running upstream, already-completed stages.
when_to_use_negative:
- When a single monolithic run is acceptable and no tool/parameter exploration is
  planned — checkpoint bookkeeping adds overhead without benefit
- When the execution environment lacks BDS's ad hoc task parallelism (the partial
  re-execution relies on BDS checkpoint support, not a generic scheduler)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: PEMA
  role: The metabarcoding pipeline whose 4-part structure emits checkpoints per step,
    enabling partial re-execution when clustering/ASV algorithms or parameters change
  repo: https://github.com/hariszaf/pema
- name: BDS
  role: BigDataScript language providing roll-back checkpoints, ad hoc task parallelism,
    task synchronization, and on-demand partial pipeline execution
- name: Docker
  role: Containerization so the checkpoint-based pipeline runs identically across
    operating systems
  repo: https://hub.docker.com/r/hariszaf/pema
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:507c1e455ac2a39250442a90e2ebec7474c0870a2ce41d6c09b72a7ce18324f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - Docker
  - PEMA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# checkpoint-based-partial-re-execution

## Summary

Implements partial re-execution of a metabarcoding pipeline by writing each stage of the PEMA workflow in BigDataScript (BDS) and relying on BDS roll-back checkpoint files, so that changing an algorithm or parameter for one step (e.g., OTU clustering vs. ASV inference) triggers re-execution of only the affected downstream steps instead of a full rerun.

## When to use

Use when building or modifying a multi-step pipeline where users may swap tools or tweak parameters (e.g., quality trimming, OTU clustering/ASV inference) and must avoid re-running upstream, already-completed stages.

## When NOT to use

- When a single monolithic run is acceptable and no tool/parameter exploration is planned — checkpoint bookkeeping adds overhead without benefit
- When the execution environment lacks BDS's ad hoc task parallelism (the partial re-execution relies on BDS checkpoint support, not a generic scheduler)

## Inputs

- .fastq.gz paired-end read files in a 'mydata' subdirectory
- parameters.tsv file (version-matched to the PEMA release) in the analysis directory
- BDS pipeline script (pema_latest.bds) split into 4 parts

## Outputs

- BDS checkpoint files enabling roll-back and partial re-execution
- Per-step subfolders containing per-analysis parameters and per-sample results
- Final (M)OTU/ASV table with taxonomy and per-sample abundances

## How to apply

Write the pipeline as a BDS script split into independent parts — Part 1 (FastQC, Trimmomatic, Cutadapt for ITS, BayesHammer, PANDAseq, obiuniq, VSEARCH chimera removal), Part 2 (VSEARCH OTU clustering for 16S/18S, CROP for ITS/COI, Swarm v2 ASV inference, user-configurable singleton removal), Part 3 (CREST LCAClassifier with Silva 128/132 and Unite, RDPClassifier with MIDORI for COI), and Part 4 (phyloseq downstream analysis) — and have each part emit checkpoint files plus per-analysis parameters into distinct self-explanatory subfolders per pipeline step (with per-sample result subfolders). BDS supports roll-back checkpoints and on-demand partial pipeline execution, so when a user switches a clustering/ASV algorithm or changes parameters in parameters.tsv, only checkpoints downstream of the modification are invalidated and re-executed. Containerize with Docker and Singularity so the checkpoint machinery works identically on HPC and personal computers. Validate by running end-to-end on the tutorial 'analysis_directory' test data and confirming checkpoints are produced and a modified step triggers partial rather than complete re-execution.

## Related tools

- **PEMA** (The metabarcoding pipeline whose 4-part structure emits checkpoints per step, enabling partial re-execution when clustering/ASV algorithms or parameters change) — https://github.com/hariszaf/pema
- **BDS** (BigDataScript language providing roll-back checkpoints, ad hoc task parallelism, task synchronization, and on-demand partial pipeline execution)
- **Docker** (Containerization so the checkpoint-based pipeline runs identically across operating systems) — https://hub.docker.com/r/hariszaf/pema

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema && ./pema_latest.bds
```

## Evaluation signals

- After a full run on the tutorial 'analysis_directory' test data, checkpoint files exist for each of the 4 pipeline parts
- Changing only a Part 2 parameter (e.g., clustering algorithm) causes re-execution of Parts 2–4 but Parts 1 outputs are reused, not recomputed
- Per-step subfolders contain both the parameters used for that step and per-sample results, matching the step that produced them
- The BDS script completes all 4 parts and returns a final (M)OTU/ASV table with taxonomy and abundances
- Re-running with an unchanged parameters.tsv reuses checkpoints without recomputation

## Limitations

- COI taxonomy assignment requires a minimum of 20 GB of RAM, which constrains where checkpointed runs can be completed
- The parameters.tsv file must match the exact PEMA version (e.g., v.2.1.4) or the checkpoint/parameter wiring will be inconsistent
- Analyses with different parameter sets require separate analysis directories, each with its own copy of parameters.tsv

## Evidence

- [abstract] BDS checkpoint support for partial re-execution: "Owing to its simplified parameterization and checkpoint support, PEMA allows users to explore alternative algorithms for specific steps of the pipeline without the need of a complete re-execution."
- [readme] BDS language and checkpoint features: [evidence span withheld — anchor status: not_found]
- [intro] Partial re-execution motivation from existing pipeline gaps: "Among the most prominent are technical difficulties in installation and use, strict limitations in setting parameters for the algorithms invoked, and incompetence in partial re-execution"
- [methods] Checkpoint and parameter bookkeeping per step: [evidence span withheld — anchor status: not_found]
