---
name: workflow-runtime-profiling
description: Use when quantifying execution cost of an amplicon pipeline run — e.g.,
  scaling behavior with core count or per-step runtime of qaqc, repseqs, taxonomy,
  and report steps.
when_to_use_negative:
- You need to compare biological results (ASV composition, taxonomy assignments) rather
  than execution performance — use a workflow run comparison instead
- The input is already a feature table or downstream artifact; runtime profiling applies
  to the raw-FASTQ-to-report pipeline execution
- The workflow does not expose a core-count parameter or step-wise entry points (tourmaline.sh
  --step / Snakemake rules)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Tourmaline
  role: Amplicon workflow being benchmarked; steps invoked via tourmaline.sh with
    --step and --cores flags
  repo: https://github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system that executes the rules and honors the --cores
    setting for timing measurements
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the underlying denoising (DADA2), taxonomy (classify-sklearn), and diversity
    steps being timed
- name: Docker
  role: Container in which the Tourmaline workflow is run for the benchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:6ebabd15c5ebda39d67cfb02f6d2c3f473fcf6ff0d6e49d7cb852d1e04bfceb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Docker
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# workflow-runtime-profiling

## Summary

Benchmarks the wall-clock runtime of a containerized amplicon analysis workflow (Tourmaline/QIIME 2 via Snakemake) by timing each main step separately at different core counts, e.g. 1 vs. 8 cores on the Lake Erie 16S rRNA dataset (PRJNA679730).

## When to use

Use when quantifying execution cost of an amplicon pipeline run — e.g., scaling behavior with core count or per-step runtime of qaqc, repseqs, taxonomy, and report steps.

## When NOT to use

- You need to compare biological results (ASV composition, taxonomy assignments) rather than execution performance — use a workflow run comparison instead
- The input is already a feature table or downstream artifact; runtime profiling applies to the raw-FASTQ-to-report pipeline execution
- The workflow does not expose a core-count parameter or step-wise entry points (tourmaline.sh --step / Snakemake rules)

## Inputs

- Raw paired-end FASTQ files (e.g. PRJNA679730, ~96 samples, ~11.5 million paired reads)
- Sample manifest file
- Sample metadata file
- Reference taxonomy database (e.g. silva-138_1)
- Three step config files (qaqc, repseqs, taxonomy)

## Outputs

- Per-step wall-clock timings (qaqc, repseqs, taxonomy, diversity/report)
- Total runtime per core count (1 core vs. 8 cores)
- Compiled timings table (runtimes.csv)

## How to apply

Set up the Tourmaline workflow (Docker container or native install) with the raw FASTQ files, sample manifest, metadata, reference database (e.g. SILVA 138.1), and the three step config files (qaqc, repseqs, taxonomy) in the working directory. Run each workflow step alone through tourmaline.sh/Snakemake — first denoising (DADA2, dada2pe), then taxonomy assignment (naive Bayes classify-sklearn), then diversity analysis (alpha rarefaction, core metrics) and HTML report generation — each with --cores 1, recording wall-clock time per step. Re-run the same unfiltered workflow end-to-end with --cores 8 to measure multi-core scaling. Compile per-step and total timings into a table such as runtimes.csv; on the ~96-sample Lake Erie dataset (~11.5 million paired reads), the reference expectation is ~5 hours at 1 core versus ~2 hours at 8 cores.

## Related tools

- **Tourmaline** (Amplicon workflow being benchmarked; steps invoked via tourmaline.sh with --step and --cores flags) — https://github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system that executes the rules and honors the --cores setting for timing measurements) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the underlying denoising (DADA2), taxonomy (classify-sklearn), and diversity steps being timed)
- **Docker** (Container in which the Tourmaline workflow is run for the benchmarking)

## Examples

```
./tourmaline.sh --step qaqc --configfile config_qaqc --cores 1
```

## Evaluation signals

- Total 1-core runtime approximates the reported ~5 hours for the full Lake Erie dataset (~96 samples, ~11.5M paired reads)
- Total 8-core runtime approximates the reported ~2 hours, confirming multi-core speedup
- Sum of individually timed steps (qaqc, repseqs, taxonomy, diversity/report) is consistent with the end-to-end runtime
- Each step is launched with the intended --cores value and its wall-clock time is logged separately
- Timings are compiled into runtimes.csv with one row per step and core count

## Limitations

- Absolute timings depend on the host machine, Docker overhead, and dataset size; only relative scaling (e.g. ~2.5x speedup from 1 to 8 cores) generalizes
- Runtimes were measured on an unfiltered workflow; representative sequence filtering parameters (repseq_min_length, repseq_min_abundance, etc., all 0 by default) affect downstream step cost
- Paper context reports no changelog found for version differences, so v1 vs. v2 runtime behavior may differ

## Evidence

- [readme] Instead of interacting with Snakemake rules directly, the main way to run Tourmaline 2 is through the `tourmaline.sh` script.: "Instead of interacting with Snakemake rules directly, the main way to run Tourmaline 2 is through the `tourmaline.sh` script."
- [readme] This script allows you to run one or more of the workflow steps at a time, specify specific config files, and set the maximum number of cores.: "This script allows you to run one or more of the workflow steps at a time, specify specific config files, and set the maximum number of cores."
- [abstract] Features include support for multiple cores, automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID): "Features include support for multiple cores, automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID)"
- [abstract] The workflow runs natively on Linux and macOS or via a Docker container: "The workflow runs natively on Linux and macOS or via a Docker container"
- [readme] ./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
