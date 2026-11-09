---
name: docker-containerized-pipeline-execution
description: Use when you need wall-clock runtime benchmarks of a full amplicon workflow
  (e.g. ~96-sample 16S rRNA dataset) under different core counts (1 vs 8 cores) inside
  the Tourmaline Docker container.
when_to_use_negative:
- Your goal is to change denoising or filtering parameters (e.g. DADA2 truncation
  lengths, repseq filters) rather than to measure runtime — use parameter-comparison
  runs instead.
- Your data is single-end or not Illumina amplicon data — Tourmaline 2 targets Illumina
  paired-end amplicon sequences.
- You need per-rule Snakemake customization beyond the three steps (qaqc, repseqs,
  taxonomy) exposed by tourmaline.sh.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Tourmaline
  role: The containerized amplicon workflow whose steps (qaqc, repseqs, taxonomy)
    are executed and timed
  repo: github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system that orchestrates the QIIME 2 commands and accepts
    the --cores setting
- name: Docker
  role: Container runtime in which the Tourmaline workflow is executed for reproducible
    benchmarking
- name: QIIME 2
  role: Underlying package that runs DADA2 denoising, classify-sklearn taxonomy, and
    diversity analyses
- name: DADA2
  role: Paired-end denoising algorithm (dada2pe) whose wall-clock time is measured
    as the denoising step
  repo: github.com/benjjneb/dada2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:059f9e164564ce82e8a8620508b9b03e1b040d403e3a09d28b6290c022410282
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Docker
  - QIIME 2
  - Snakemake
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# docker-containerized-pipeline-execution

## Summary

Runs the Tourmaline amplicon workflow (QIIME 2 + Snakemake) inside its Docker container via tourmaline.sh, executing the qaqc, repseqs, and taxonomy steps with a specified core count and timing each step, as demonstrated on the 2018 Lake Erie 16S rRNA dataset (PRJNA679730).

## When to use

Use when you need wall-clock runtime benchmarks of a full amplicon workflow (e.g. ~96-sample 16S rRNA dataset) under different core counts (1 vs 8 cores) inside the Tourmaline Docker container.

## When NOT to use

- Your goal is to change denoising or filtering parameters (e.g. DADA2 truncation lengths, repseq filters) rather than to measure runtime — use parameter-comparison runs instead.
- Your data is single-end or not Illumina amplicon data — Tourmaline 2 targets Illumina paired-end amplicon sequences.
- You need per-rule Snakemake customization beyond the three steps (qaqc, repseqs, taxonomy) exposed by tourmaline.sh.

## Inputs

- Raw paired FASTQ files (Lake Erie 16S rRNA dataset PRJNA679730, ~96 samples, ~11.5 million paired reads)
- Sample manifest
- Sample metadata file
- Reference database (e.g. SILVA 138.1)
- Three step config files: qaqc, repseqs, taxonomy

## Outputs

- Per-step wall-clock timings for denoising (DADA2 dada2pe), taxonomy (classify-sklearn), diversity analysis (alpha rarefaction, core metrics), and HTML report generation
- Total runtime for 1-core and 8-core runs
- runtimes.csv compiling per-step and total timings

## How to apply

Set up the Tourmaline workflow inside its Docker container and place the raw paired FASTQ files, sample manifest, metadata, reference database, and the three step config files (qaqc, repseqs, taxonomy) in the working directory. Run each main workflow step separately through tourmaline.sh/Snakemake with --cores 1 — DADA2 paired-end denoising (dada2pe), naive Bayes classify-sklearn taxonomy assignment, alpha rarefaction/core metrics diversity analysis, and HTML report generation — recording each step's wall-clock time. Then re-run the same unfiltered workflow end-to-end with --cores 8 to measure multi-core scaling. Tourmaline 2 exposes execution through the tourmaline.sh script rather than direct Snakemake rule calls, and each of the three steps has its own Snakefile. For the Lake Erie dataset (~96 samples, ~11.5 million paired reads), the expected result is roughly 5 hours on 1 core and roughly 2 hours on 8 cores.

## Related tools

- **Tourmaline** (The containerized amplicon workflow whose steps (qaqc, repseqs, taxonomy) are executed and timed) — github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system that orchestrates the QIIME 2 commands and accepts the --cores setting)
- **Docker** (Container runtime in which the Tourmaline workflow is executed for reproducible benchmarking)
- **QIIME 2** (Underlying package that runs DADA2 denoising, classify-sklearn taxonomy, and diversity analyses)
- **DADA2** (Paired-end denoising algorithm (dada2pe) whose wall-clock time is measured as the denoising step) — github.com/benjjneb/dada2

## Examples

```
./tourmaline.sh --step qaqc,repseqs,taxonomy --configfile config1,config2,config3 --cores 8
```

## Evaluation signals

- The 1-core total runtime for the full Lake Erie dataset approximates the reported ~5 hours.
- The 8-core end-to-end runtime approximates the reported ~2 hours, confirming multi-core speedup.
- Each of the three steps (qaqc, repseqs, taxonomy) completes via tourmaline.sh with its own Snakefile and a recorded wall-clock time.
- Sum of individually timed steps is consistent with the end-to-end total in runtimes.csv.
- The workflow completes inside the Docker container with the manifest, metadata, reference database, and three config files in place.

## Limitations

- Runtime figures are hardware-dependent; the ~5 h (1 core) vs ~2 h (8 cores) results were obtained for ~96 samples / ~11.5 million paired reads and may not transfer to other datasets or machines.
- tourmaline.sh must be run from within the tourmaline directory.
- The reported timings are for an unfiltered workflow; representative sequence filtering (taxonomy, length, abundance, prevalence, or ID) would change the compute profile.

## Evidence

- [abstract] Tourmaline runs natively on Linux and macOS or via a Docker container: "The workflow runs natively on Linux and macOS or via a Docker container"
- [abstract] Multi-core support is a stated feature of Tourmaline: "Features include support for multiple cores, automatic determination of trimming parameters using quality scores"
- [readme] tourmaline.sh is the main entry point for running workflow steps with core limits: "Instead of interacting with Snakemake rules directly, the main way to run Tourmaline 2 is through the `tourmaline.sh` script."
- [readme] tourmaline.sh exposes step selection, config files, and core count: "This script allows you to run one or more of the workflow steps at a time, specify specific config files, and set the maximum number of cores."
- [readme] The three workflow steps have their own Snakefiles: "Each of the three steps (explained more below) has its own Snakefile, so you must specify the correct snakefile when running an individual rule."
