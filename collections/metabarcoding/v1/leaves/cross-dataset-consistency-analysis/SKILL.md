---
name: cross-dataset-consistency-analysis
description: Use when differential abundance results exist for the same phenotype
  across multiple independent datasets (e.g., five diarrhea or five obesity studies)
  and you need to quantify which DA tools call the same genera significant more often
  than expected by chance.
when_to_use_negative:
- The datasets span different environments/phenotypes where consistency could reflect
  technical or biological factors rather than shared signal — the article cautions
  that 'not all environments and datasets are appropriate for this comparison' (in
  the obesity group most tools were no better than chance).
- The goal is false-positive rate estimation under randomized group labels rather
  than cross-study consistency — use run_all_FP_datasets_filt.sh / run_all_FP_datasets_unfilt.sh
  instead of the consistency drivers.
- Only one dataset is available per phenotype, so there is no cross-study overlap
  to count; use within-dataset consensus/intersect of tool outputs instead.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: ALDEx2
  role: DA tool whose significant-genus calls are compared across the five diarrhea/obesity
    datasets; showed the highest consistency (1.353-fold, p<0.001); run at pinned
    version 1.18.0.
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ANCOM-II
  role: DA tool evaluated for cross-study consistency; performed better than average
    at identifying the same genera as significantly DA across the five diarrhea datasets.
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: edgeR
  role: Negative-binomial DA tool included as a consistency comparator (fold diff
    1.105, p=0.002); run at pinned version 3.28.1 via run_all_tools.sh.
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: DESeq2
  role: Negative-binomial DA tool generating per-dataset significant genus calls for
    the overlap counts; run at pinned version 1.26.0.
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: metagenomeSeq
  role: Zero-inflated Gaussian DA tool generating per-dataset significant genus calls;
    run at pinned version 1.28.2.
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:1e12cd03603de2a469cf5359a4f598bd41be8430efc0a350587d832cd3973397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-II
  - DESeq2
  - LEfSe
  - corncob
  - dplyr
  - edgeR
  - limma voom
  - metagenomeSeq
  - tidyverse
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# cross-dataset-consistency-analysis

## Summary

Quantifies how consistently a differential abundance tool calls the same genera significant across multiple independent datasets of one phenotype (five diarrhea or five obesity 16S rRNA datasets), comparing observed cross-study overlap against a permutation-based random expectation and reporting observed/expected fold differences (e.g., ALDEx2 1.353-fold, p<0.001 in diarrhea).

## When to use

Use when differential abundance results exist for the same phenotype across multiple independent datasets (e.g., five diarrhea or five obesity studies) and you need to quantify which DA tools call the same genera significant more often than expected by chance.

## When NOT to use

- The datasets span different environments/phenotypes where consistency could reflect technical or biological factors rather than shared signal — the article cautions that 'not all environments and datasets are appropriate for this comparison' (in the obesity group most tools were no better than chance).
- The goal is false-positive rate estimation under randomized group labels rather than cross-study consistency — use run_all_FP_datasets_filt.sh / run_all_FP_datasets_unfilt.sh instead of the consistency drivers.
- Only one dataset is available per phenotype, so there is no cross-study overlap to count; use within-dataset consensus/intersect of tool outputs instead.

## Inputs

- Genus-level collapsed 16S rRNA ASV count tables (TSV) for the five diarrhea datasets (GEMS, dia_schneider, Human-C. diff 1, Human-C. diff 2, Human-Inf.) and the five obesity datasets
- Two-column metadata TSV per dataset (first column sample names, second column sample groupings, maximum 2 groupings)
- config.sh with DATA_DIR, ANCOM_DIR, and TOOL_DIR paths pointing to the unzipped figshare Hackathon data and the repo's Tool_scripts/Ancom2_Script folders
- Raw 16S rRNA Microbiome Datasets archive from figshare (https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724)

## Outputs

- Per-tool counts of genera called significantly differentially abundant in common across all five datasets of a phenotype group
- Per-tool fold difference of observed vs. expected consistency with permutation p-value (Table 2-style values, e.g., ALDEx2 1.353 at p<0.001)
- Output CSV containing one fold-difference row per DA tool (14 tools)
- Intermediate RDS files for the genera consistency analysis (Misc_datafiles/consistency_analysis_RDS_out)

## How to apply

Download the raw 16S rRNA datasets from figshare, set DATA_DIR, ANCOM_DIR, and TOOL_DIR in config.sh, and run under R 3.6.3 with pinned package versions (ALDEx2 1.18.0, DESeq2 1.26.0, edgeR 3.28.1, metagenomeSeq 1.28.2) or the dockerdkd/hackathon2021 Docker image. Collapse each ASV table to genus level for the five diarrhea datasets (GEMS, dia_schneider, Human-C. diff 1, Human-C. diff 2, Human-Inf.) and the five obesity datasets, then execute run_all_tools.sh on each collapsed dataset via the run_diarrhea_*.sh and run_obesity_*.sh driver scripts with rarefaction depth -D 2000 and 10% prevalence filtering -F 0.1. For each of the 14 DA tools, count the observed number of genera called significantly differentially abundant in common across all five datasets within each phenotype group (intermediate results are kept as RDS files in Misc_datafiles/consistency_analysis_RDS_out). Compute the expected overlap under a permutation null that assumes all genera are equally likely to be significant by chance, then report each tool's observed/expected fold difference and significance (Table 2 references: ALDEx2 1.353 at p<0.001, edgeR 1.105 at p=0.002). Because this null ignores prevalence differences among genera, treat fold differences as a relative ranking of tool consistency — ALDEx2 and ANCOM-II were the most consistent — rather than an absolute reproducibility measure.

## Related tools

- **ALDEx2** (DA tool whose significant-genus calls are compared across the five diarrhea/obesity datasets; showed the highest consistency (1.353-fold, p<0.001); run at pinned version 1.18.0.) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ANCOM-II** (DA tool evaluated for cross-study consistency; performed better than average at identifying the same genera as significantly DA across the five diarrhea datasets.) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **edgeR** (Negative-binomial DA tool included as a consistency comparator (fold diff 1.105, p=0.002); run at pinned version 3.28.1 via run_all_tools.sh.) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **DESeq2** (Negative-binomial DA tool generating per-dataset significant genus calls for the overlap counts; run at pinned version 1.26.0.) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **metagenomeSeq** (Zero-inflated Gaussian DA tool generating per-dataset significant genus calls; run at pinned version 1.28.2.) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Computed fold differences match the Table 2 reference values: ALDEx2 1.353 at p<0.001 and edgeR 1.105 at p=0.002 for the diarrhea datasets.
- The output CSV contains exactly one fold-difference row per DA tool (14 tools).
- Obesity-group fold differences cluster near 1 — consistency no higher than expected by chance for most tools.
- ALDEx2 and ANCOM-II emerge as the most consistent tools across studies, matching the article's abstract-level finding.
- Before the consistency runs, reproduce the expected Human-ALL (MALL) output in Misc_datafiles/Expected_MALL_output (~20 min on a modern CPU, single core) to confirm pinned R 3.6.3 tool versions are working.

## Limitations

- The permutation null assumes all genera are equally likely to be significant by chance, which is invalid due to prevalence differences, so the random expectation is quite simplistic.
- Not all environments and datasets are appropriate for this cross-data comparison; obesity-group consistency could reflect technical or biological factors differing between datasets.
- Cross-study consistency does not guarantee reliability — there is no assurance that the intersection of tool outputs is more reliable, as tools may simply pick up the same noise as significant.
- Compute-bound: a single dataset takes ~20 minutes on a modern CPU single core, and all tools must be installed (or the dockerdkd/hackathon2021 image used) with the pinned R 3.6.3 package versions.

## Evidence

- [abstract] ALDEx2 and ANCOM-II most consistent: "ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches"
- [discussion] ANCOM-II cross-dataset performance: "ANCOM-II performed better than average at identifying the same genera as significantly DA across five diarrhea-related datasets"
- [discussion] Obesity group near-chance consistency: "the consistency of significant genera across five datasets comparing obese and control individuals was no higher than expected by chance for most tools."
- [discussion] Simplistic permutation null caveat: [evidence span withheld — anchor status: not_found]
- [readme] Diarrhea consistency driver scripts: "run_diarrhea_*.sh --> runs the diarrhea datasets for the consistency across datasets analysis"
- [readme] Consistency intermediate files: "Intermediate files for genera consistency analysis (_consistency_analysis_RDS_out_)"
- [readme] Prevalence filter parameter definition: "0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
