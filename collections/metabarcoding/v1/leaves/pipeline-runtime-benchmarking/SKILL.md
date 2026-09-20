---
name: pipeline-runtime-benchmarking
description: Use when quantifying how a configured dadasnake/Snakemake workflow's
  wall-clock time scales with core count (e.g. -t 4 vs -t 15) on a defined dataset
  such as a 267-sample ITS1 run with 46.8 million reads.
when_to_use_negative:
- The analysis goal is ASV accuracy or taxonomy quality rather than compute scaling
  — use mock community benchmarking instead
- The dataset is a small mock community whose runtime is too short for a meaningful
  4-vs-15-core comparison
- The workflow is not Snakemake-based, since the parallelization and speedup claims
  rely on Snakemake-triggered jobs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: dadasnake
  role: 1-command Snakemake pipeline whose runtime is being benchmarked; invoked with
    -d (dryrun) and -t N (thread cap)
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Workflow engine that provides the parallelization responsible for the ~2x
    speedup at 15 cores
- name: conda
  role: Provides the single conda environment in which dadasnake and its dependencies
    (DADA2, cutadapt, ITSx, mothur) run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:c2e3ed843607471857afa6d2a2fd2307fabed251f2b6b2269cce6b99a2acd45b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pipeline-runtime-benchmarking

## Summary

Benchmark wall-clock runtime of a Snakemake-based amplicon pipeline (dadasnake) by executing the identical workflow twice with different thread caps and recording the scaling factor. Motivated by dadasnake's design goal of efficient use of high-performance computing infrastructures.

## When to use

Use when quantifying how a configured dadasnake/Snakemake workflow's wall-clock time scales with core count (e.g. -t 4 vs -t 15) on a defined dataset such as a 267-sample ITS1 run with 46.8 million reads.

## When NOT to use

- The analysis goal is ASV accuracy or taxonomy quality rather than compute scaling — use mock community benchmarking instead
- The dataset is a small mock community whose runtime is too short for a meaningful 4-vs-15-core comparison
- The workflow is not Snakemake-based, since the parallelization and speedup claims rely on Snakemake-triggered jobs

## Inputs

- Raw paired-end ITS1 fastq files for 267 samples (NCBI SRA BioProject PRJNA272747, 46.8 million reads)
- Tab-separated sample table with columns 'library', 'r1_file', 'r2_file'
- dadasnake config.yaml (paired=true, ITSx region=ITS1, mothur classification vs UNITE-formatted fungal ITS database, pair_merging min_overlap >= 20 bp)
- Compute allocation: four 8 GB cores (first run) and 15 cores (second run)

## Outputs

- runtime_benchmark.tsv with columns threads and wall-clock hours for each run
- workflow.done marker file in the output directory per completed run
- Full dadasnake outputs per run: ASV/OTU tables, intermediary figures and statistics (Fig. 2c and d analogues)

## How to apply

Retrieve the raw paired-end fastq files (e.g. the 267-sample, 46.8-million-read ITS1 dataset from NCBI SRA BioProject PRJNA272747) and build a tab-separated sample table with 'library', 'r1_file', and 'r2_file' headers. Write a dadasnake config.yaml with paired=true, ITSx enabled with region set to ITS1, mothur-based Bayesian classification against a fungal ITS (UNITE-formatted) database, and pair_merging min_overlap of at least 20 bp. Validate the setup with a dryrun (dadasnake -d) to confirm input files, sample table formatting, and configuration before committing compute. Execute dadasnake with the thread cap set to 4 (-t 4) on four 8 GB cores, running primer removal (cutadapt), quality filtering and trimming, error estimation and ASV inference (DADA2), read-pair merging, chimera removal, ITSx filtering, and mothur classification, recording total wall-clock time (expected just under 4 hours). Re-execute the identical workflow with -t 15 and record the wall-clock time, expecting roughly a 2-fold runtime reduction from Snakemake parallelization. Write both runtimes (threads, wall-clock hours) to runtime_benchmark.tsv.

## Related tools

- **dadasnake** (1-command Snakemake pipeline whose runtime is being benchmarked; invoked with -d (dryrun) and -t N (thread cap)) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Workflow engine that provides the parallelization responsible for the ~2x speedup at 15 cores)
- **conda** (Provides the single conda environment in which dadasnake and its dependencies (DADA2, cutadapt, ITSx, mothur) run)

## Examples

```
dadasnake -d; dadasnake -t 4; dadasnake -t 15
```

## Evaluation signals

- Each run completes, verified by the 'workflow.done' marker file in the output directory
- First run (4 threads, four 8 GB cores) finishes in just under 4 hours for the 267-sample, 46.8-million-read ITS1 dataset
- Second run (15 threads) shows approximately a 2-fold runtime reduction relative to the 4-thread run
- runtime_benchmark.tsv contains both rows with plausible (threads, wall-clock hours) pairs consistent with the reported benchmark
- Dryrun (dadasnake -d) passes with no errors in sample table formatting or configuration before either timed run

## Limitations

- Speedup is sublinear (factor of ~2 for ~3.75x more cores), so perfect linear scaling should not be expected
- Runtime results are specific to dataset size and core memory (8 GB cores); other datasets or node types will differ
- Snakemake version compatibility: dadasnake has been tested with Snakemake >= 5.9.1 and not with previous versions

## Evidence

- [results] dadasnake processed the 267-sample, 46.8-million-read ITS1 dataset in just under 4 hours on four 8 GB cores, and using 15 cores reduced the runtime by a factor of 2 (Fig. 2c and d).: [evidence span withheld — anchor status: not_found]
- [abstract] Snakemake efficiency finding: "By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures."
- [intro] Workflow structure quote: "It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake"
- [readme] README thread-cap quote: "MAX_THREADS - set this to the maximum number of cores you want to be using in a run. If you don't set this, the default will be 50. Users can override this setting at runtime."
- [readme] README dryrun/completeness basis: "have [tmux](https://github.com/tmux/tmux/wiki) installed"
