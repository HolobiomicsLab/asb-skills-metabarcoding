---
name: 16s-rrna-metabarcoding-pipeline-benchmarking
description: Use when you have raw 16S rRNA .fastq reads and need to compare a new
  metabarcoding pipeline's OTU inference, runtime, and community-composition results
  against established pipelines.
when_to_use_negative:
- Input is already an OTU/ASV feature table rather than raw .fastq.gz reads — pipeline-level
  benchmarking requires running each pipeline from raw reads.
- The goal is only taxonomic assignment quality (e.g., mock-community F1-score evaluation)
  rather than cross-pipeline comparison of community composition.
- Data are not 16S/18S rRNA amplicons with paired-end structure amenable to PANDAseq
  assembly in PEMA.
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: PEMA
  role: Pipeline under evaluation; runs read pre-processing, OTU clustering/ASV inference,
    and taxonomy assignment, and provides the phyloseq R script for downstream analysis
  repo: https://github.com/hariszaf/pema
- name: LotuS
  role: Established comparison pipeline whose OTU table feeds the PERMANOVA cross-pipeline
    comparison
- name: QIIME 2
  role: Established comparison pipeline run in DADA2 and Deblur variants for OTU counts
    and community-composition comparison
- name: mothur
  role: Established comparison pipeline run in single-thread mode with documented
    parameter sets
- name: phyloseq
  role: R package (invoked via PEMA's phyloseq_in_PEMA.R) used to compute taxonomic
    profiles and beta-diversity before PERMANOVA
- name: BDS
  role: BigDataScript language implementing PEMA; provides ad hoc task parallelism,
    roll-back checkpoints, and on-demand partial execution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:c5b35be1fb9d18e6a081ef8e686ccc2ad5c0271b07761a48cb47f20e90f20f4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - LotuS
  - PEMA
  - QIIME 2
  - mothur
  - phyloseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# 16S rRNA metabarcoding pipeline benchmarking

## Summary

Benchmark PEMA's 16S rRNA metabarcoding output against established pipelines (LotuS, mothur, QIIME 2 with DADA2/Deblur) by comparing OTU tables, execution time, and beta-diversity via PERMANOVA on a salinity-gradient dataset.

## When to use

Use when you have raw 16S rRNA .fastq reads and need to compare a new metabarcoding pipeline's OTU inference, runtime, and community-composition results against established pipelines.

## When NOT to use

- Input is already an OTU/ASV feature table rather than raw .fastq.gz reads — pipeline-level benchmarking requires running each pipeline from raw reads.
- The goal is only taxonomic assignment quality (e.g., mock-community F1-score evaluation) rather than cross-pipeline comparison of community composition.
- Data are not 16S/18S rRNA amplicons with paired-end structure amenable to PANDAseq assembly in PEMA.

## Inputs

- 16S rRNA paired-end .fastq.gz read files downloaded from ENA
- PEMA parameters.tsv matched to the PEMA version being run
- metadata.csv (comma-separated) for phyloseq downstream analysis
- OTU tables produced by each pipeline (PEMA, LotuS, DADA2, mothur, QIIME 2 variants)

## Outputs

- per-sample OTU counts and execution times per pipeline
- taxonomic profiles and beta-diversity results from the phyloseq R script
- PERMANOVA statistics comparing community compositions (e.g., permanova_comparison.tsv)

## How to apply

Download the 16S rRNA .fastq read files from ENA (e.g., with ENA File Downloader) and run PEMA in its Singularity or Docker container, exploiting BDS intrinsic parallelization across all available cores (e.g., 20 Intel Xeon 2.60 GHz cores on Dell M630 HPC nodes with 128 GB RAM). Process the same reads with the comparison pipelines (LotuS, mothur, QIIME 2 with DADA2 and Deblur variants) in single-thread mode using the parameter sets documented in the paper's Additional File 1, recording execution time and per-sample OTU counts. Analyze all OTU tables with the phyloseq R script shipped in PEMA's analysis_directory to compute taxonomic profiles and alpha-/beta-diversity. Run PERMANOVA on the PEMA, LotuS, and DADA2 community compositions to test habitat differentiation among river, lagoon, and open sea samples, and check that per-sample OTU numbers decrease with increasing salinity.

## Related tools

- **PEMA** (Pipeline under evaluation; runs read pre-processing, OTU clustering/ASV inference, and taxonomy assignment, and provides the phyloseq R script for downstream analysis) — https://github.com/hariszaf/pema
- **LotuS** (Established comparison pipeline whose OTU table feeds the PERMANOVA cross-pipeline comparison)
- **QIIME 2** (Established comparison pipeline run in DADA2 and Deblur variants for OTU counts and community-composition comparison)
- **mothur** (Established comparison pipeline run in single-thread mode with documented parameter sets)
- **phyloseq** (R package (invoked via PEMA's phyloseq_in_PEMA.R) used to compute taxonomic profiles and beta-diversity before PERMANOVA)
- **BDS** (BigDataScript language implementing PEMA; provides ad hoc task parallelism, roll-back checkpoints, and on-demand partial execution)

## Examples

```
docker run -it -v /path/to/analysis_directory/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- PERMANOVA for PEMA results reproduces F.Model = 7.0718, P < 0.001 and for LotuS F.Model = 6.5901, P < 0.001 on the Pavloudi et al. salinity-gradient dataset
- Per-sample OTU numbers show a decreasing trend with increasing salinity across river, lagoon, and open sea samples
- Execution time and OTU counts are recorded for every pipeline (PEMA, LotuS, mothur, QIIME 2 DADA2/Deblur) under the documented parameter sets
- All statistics from the comparison are written to a tabular output such as permanova_comparison.tsv

## Limitations

- Comparison pipelines (LotuS, mothur, QIIME 2) were run in single-thread mode while PEMA exploited BDS parallelization across all 20 cores, so runtime comparisons are not compute-equalized.
- The parameter sets used for the comparison pipelines are only documented in Additional File 1, which is not included in the article text.
- PEMA's COI taxonomy assignment requires a minimum of 20 GB of RAM, and results depend on the matched parameters.tsv version for the PEMA release used.

## Evidence

- [methods] PERMANOVA was run for PEMA, LotuS and DADA2 results: "when the PERMANOVA analysis was run for the results of PEMA, LotuS, and DADA2"
- [abstract] PEMA evaluated against published datasets with comparable quality: "PEMA was evaluated against both mock communities and previously published datasets and achieved results of comparable quality."
- [readme] BDS provides parallelism and checkpoints in PEMA: [evidence span withheld — anchor status: not_found]
- [readme] PEMA supports VSEARCH OTU clustering and Swarm ASV inference: [evidence span withheld — anchor status: not_found]
