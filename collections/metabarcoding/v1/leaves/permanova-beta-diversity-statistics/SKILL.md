---
name: permanova-beta-diversity-statistics
description: Use when comparing community compositions output by different metabarcoding
  pipelines (or treatments) and you need a formal hypothesis test of habitat/sample-group
  differentiation plus a richness-vs-environmental-gradient check.
when_to_use_negative:
- Input is already an ecological/feature table and the goal is only abundance summarization
  — this skill is for statistical comparison of community composition across pipelines/groups,
  not descriptive profiling.
- You only have single-pipeline results and no comparison set (e.g. PEMA results alone)
  — PERMANOVA comparison requires OTU tables from at least PEMA, LotuS, and DADA2
  as in the study.
- Your data are not habitat-replicated community profiles (e.g. a single sample or
  non-ecological amplicon data) — no group structure exists for PERMANOVA.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: PEMA
  role: Source of the OTU tables under test and of the phyloseq R script used for
    downstream beta-diversity and statistical analysis
  repo: https://github.com/hariszaf/pema
- name: phyloseq
  role: R package invoked (via phyloseq_in_PEMA.R) to compute taxonomic profiles and
    beta-diversity from the OTU tables
  repo: http://joey711.github.io/phyloseq/index.html
- name: vegan
  role: R package providing certain downstream statistical functions executed alongside
    phyloseq in PEMA's ecological analysis step
- name: LotuS
  role: Established pipeline whose OTU/community-composition results are compared
    against PEMA's in the PERMANOVA analysis
- name: DADA2
  role: Established pipeline (including its QIIME 2 variant) whose results are included
    in the PERMANOVA comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:26b311bbe9d7d8de5ddc398864c245dd7891a005687f155aea6b587bc4f9e6d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - DADA2
  - LotuS
  - PEMA
  - QIIME 2
  - mothur
  - phyloseq
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# permanova-beta-diversity-statistics

## Summary

Beta-diversity and PERMANOVA-based statistical comparison of metabarcoding OTU tables (PEMA, LotuS, DADA2) on a 16S salinity-gradient dataset, using the phyloseq R package to test whether community composition differentiates across habitats and whether OTU richness declines with salinity.

## When to use

Use when comparing community compositions output by different metabarcoding pipelines (or treatments) and you need a formal hypothesis test of habitat/sample-group differentiation plus a richness-vs-environmental-gradient check.

## When NOT to use

- Input is already an ecological/feature table and the goal is only abundance summarization — this skill is for statistical comparison of community composition across pipelines/groups, not descriptive profiling.
- You only have single-pipeline results and no comparison set (e.g. PEMA results alone) — PERMANOVA comparison requires OTU tables from at least PEMA, LotuS, and DADA2 as in the study.
- Your data are not habitat-replicated community profiles (e.g. a single sample or non-ecological amplicon data) — no group structure exists for PERMANOVA.

## Inputs

- OTU tables (sample-by-OTU abundance matrices) from PEMA, LotuS, and DADA2 on the Pavloudi et al. 16S salinity-gradient reads
- metadata.csv mapping samples to habitats (river, lagoon, open sea) and salinity values, in comma-separated format
- phyloseq_in_PEMA.R analysis script from the PEMA analysis_directory

## Outputs

- permanova_comparison.tsv with F.Model and P values per pipeline
- per-sample OTU counts tabulated against salinity
- beta-diversity and taxonomic-profile outputs from the phyloseq R script

## How to apply

Run each pipeline (PEMA, LotuS, DADA2) on the same .fastq reads, then analyze all resulting OTU tables with the phyloseq R script shipped with PEMA to compute taxonomic profiles and beta-diversity. Run PERMANOVA on the community compositions of PEMA, LotuS, and DADA2 specifically, testing habitat differentiation among river, lagoon, and open sea samples (P < 0.001 expected). Separately tabulate per-sample OTU numbers against the salinity gradient and confirm the number of OTUs decreases with increasing salinity. Write all statistics to permanova_comparison.tsv. The rationale is that a pipeline recovering biologically meaningful structure should reproduce the published F.Model values: 7.0718 for PEMA and 6.5901 for LotuS.

## Related tools

- **PEMA** (Source of the OTU tables under test and of the phyloseq R script used for downstream beta-diversity and statistical analysis) — https://github.com/hariszaf/pema
- **phyloseq** (R package invoked (via phyloseq_in_PEMA.R) to compute taxonomic profiles and beta-diversity from the OTU tables) — http://joey711.github.io/phyloseq/index.html
- **vegan** (R package providing certain downstream statistical functions executed alongside phyloseq in PEMA's ecological analysis step)
- **LotuS** (Established pipeline whose OTU/community-composition results are compared against PEMA's in the PERMANOVA analysis)
- **DADA2** (Established pipeline (including its QIIME 2 variant) whose results are included in the PERMANOVA comparison)

## Examples

```
singularity run -B /path/to/analysis_directory/:/mnt/analysis pema_v.2.1.4.sif  # then inside the container: ./pema_latest.bds with phyloseq_in_PEMA.R and metadata.csv present to emit beta-diversity/PERMANOVA outputs
```

## Evaluation signals

- PERMANOVA on PEMA results must reproduce F.Model = 7.0718 with P < 0.001
- PERMANOVA on LotuS results must reproduce F.Model = 6.5901 with P < 0.001
- Per-sample OTU numbers must show a decreasing trend with increasing salinity
- PERMANOVA must detect significant habitat differentiation among river, lagoon, and open sea samples
- All statistics must be written to permanova_comparison.tsv

## Limitations

- The published PERMANOVA comparison covers only PEMA, LotuS, and DADA2 results; mothur and QIIME 2 (Deblur variant) outputs were processed for execution time and OTU counts but not included in the PERMANOVA test as reported.
- Running the phyloseq step via PEMA requires exactly named files: phyloseq_in_PEMA.R and metadata.csv (comma-separated) in the analysis directory; PEMA fails unless naming matches.
- The parameters.tsv file must match the PEMA version used (e.g. v.2.1.4), otherwise the run will not behave as documented.

## Evidence

- [methods] PERMANOVA analysis compared PEMA with established pipelines: "when the PERMANOVA analysis was run for the results of PEMA, LotuS, and DADA2"
- [methods] phyloseq provides the downstream ecological analysis: "This includes α- and β-diversity analysis, taxonomic composition, statistical comparisons, and calculation of correlations between samples"
- [readme] README confirms phyloseq-based downstream analysis within PEMA: [evidence span withheld — anchor status: not_found]
- [readme] README confirms vegan functions also run in downstream analysis: [evidence span withheld — anchor status: not_found]
