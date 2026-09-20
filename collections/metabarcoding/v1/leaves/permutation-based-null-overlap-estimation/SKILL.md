---
name: permutation-based-null-overlap-estimation
description: Use when multiple DA tools have been run independently on several datasets
  of the same disease/environment and you need to test whether the overlap of significant
  genera exceeds chance (consistency analysis).
when_to_use_negative:
- Datasets are not from the same environment/phenotype — the article notes 'not all
  environments and datasets are appropriate for this comparison' (e.g., obesity consistency
  was no higher than chance for most tools)
- You need a rigorous null model for individual taxon significance rather than a cross-dataset
  overlap expectation; the authors flag the random expectation as 'quite simplistic'
  since it ignores prevalence differences among genera
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: ALDEx2
  role: DA tool whose cross-dataset significant-genus overlap is compared to the permutation
    null; achieved 1.35-fold consistency increase (p<0.001) on diarrhea datasets
- name: edgeR
  role: DA tool used as a lower-consistency comparator (fold diff 1.105, p=0.002)
    in the overlap analysis
- name: ANCOM
  role: Standalone ANCOM implementation (ANCOM-Code-Archive) used as one of the 14
    tools feeding significant genus lists into the overlap computation
  repo: FrederickHuangLin/ANCOM-Code-Archive
- name: DESeq2
  role: Negative-binomial DA tool run on each collapsed genus table before counting
    cross-dataset overlap
- name: metagenomeSeq
  role: Zero-inflated Gaussian DA tool (v1.28.2) run on each collapsed genus table
    before counting cross-dataset overlap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:04f31b4701bc43c831dcfb3c23a9ee0cde9fd7ccd7f775e63ccba0cdfb4ec7ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-II
  - DESeq2
  - LEfSe
  - corncob
  - edgeR
  - limma voom
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# permutation-based-null-overlap-estimation

## Summary

Quantifies cross-study consistency of differential abundance calls by comparing the observed number of significant genera shared across all datasets of the same phenotype (e.g., five diarrhea or five obesity datasets) against a permutation-based random expectation, reporting fold-difference and significance per DA tool.

## When to use

Use when multiple DA tools have been run independently on several datasets of the same disease/environment and you need to test whether the overlap of significant genera exceeds chance (consistency analysis).

## When NOT to use

- Datasets are not from the same environment/phenotype — the article notes 'not all environments and datasets are appropriate for this comparison' (e.g., obesity consistency was no higher than chance for most tools)
- You need a rigorous null model for individual taxon significance rather than a cross-dataset overlap expectation; the authors flag the random expectation as 'quite simplistic' since it ignores prevalence differences among genera

## Inputs

- Genus-level collapsed ASV/OTU count tables (TSV) for the five diarrhea and five obesity 16S rRNA datasets from figshare
- Two-column metadata TSV (sample names, sample groupings)
- Per-dataset, per-tool significant DA genus lists from run_all_tools.sh outputs

## Outputs

- Per-tool counts of genera significant in common across all five datasets per phenotype group
- Expected overlap under the permutation (random) null per tool
- Fold-difference (observed/expected) and p-value per tool, e.g., Table 2 rows: ALDEx2 1.353, p<0.001; edgeR 1.105, p=0.002; one row per DA tool in an output CSV

## How to apply

First run each of the 14 DA tools (ALDEx2, ANCOM-II, ANCOM, LEfSe, DESeq2, edgeR, corncob, metagenomeSeq, limma voom, etc.) via run_all_tools.sh on each collapsed genus-level table using rarefaction depth -D 2000 and 10% prevalence filtering -F 0.1 under R 3.6.3 with pinned versions (ALDEx2 1.18.0, DESeq2 1.26.0, edgeR 3.28.1, metagenomeSeq 1.28.2). For each tool, count the observed number of genera called significantly differentially abundant in common across all five datasets within a phenotype group (diarrhea: GEMS, dia_schneider, Human-C. diff 1, Human-C. diff 2, Human-Inf.; or the five obesity datasets). Compute the expected overlap under a permutation null that assumes all genera are equally likely to be significant by chance. For each tool, calculate the fold difference of observed vs. expected consistency and its p-value, reproducing Table 2 values (e.g., ALDEx2 fold diff 1.353, p<0.001; edgeR fold diff 1.105, p=0.002). The drivers run_diarrhea_*.sh and run_obesity_*.sh execute this consistency-across-datasets analysis.

## Related tools

- **ALDEx2** (DA tool whose cross-dataset significant-genus overlap is compared to the permutation null; achieved 1.35-fold consistency increase (p<0.001) on diarrhea datasets)
- **edgeR** (DA tool used as a lower-consistency comparator (fold diff 1.105, p=0.002) in the overlap analysis)
- **ANCOM** (Standalone ANCOM implementation (ANCOM-Code-Archive) used as one of the 14 tools feeding significant genus lists into the overlap computation) — FrederickHuangLin/ANCOM-Code-Archive
- **DESeq2** (Negative-binomial DA tool run on each collapsed genus table before counting cross-dataset overlap)
- **metagenomeSeq** (Zero-inflated Gaussian DA tool (v1.28.2) run on each collapsed genus table before counting cross-dataset overlap)

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Computed fold differences and p-values match the Table 2 reference values (ALDEx2 1.353 at p<0.001; edgeR 1.105 at p=0.002)
- Output CSV contains exactly one fold-difference row per DA tool (14 tools)
- Observed overlap counts are integers ≤ the minimum number of significant genera called by that tool in any single dataset
- Re-running the pipeline with the pinned R 3.6.3 package versions and -D 2000 / -F 0.1 settings reproduces the counts in the consistency_analysis_RDS_out intermediate files
- Permutation expected overlaps are reproducible given the same random seed/permutation count

## Limitations

- The random expectation is simplistic: it assumes all genera are equally likely to be significant by chance, which is invalid due to prevalence differences
- Not all environments/datasets are appropriate for this comparison; obesity dataset consistency was no higher than expected by chance for most tools and may reflect technical/biological differences between datasets
- ALDEx2's higher consistency (like the consensus intersection) does not guarantee correctness — tools could be picking up the same noise as significant

## Evidence

- [discussion] ANCOM-II performed better than average at identifying the same genera as significantly DA across five diarrhea-related datasets: "ANCOM-II performed better than average at identifying the same genera as significantly DA across five diarrhea-related datasets"
- [discussion] Simplistic random expectation caveat: [evidence span withheld — anchor status: not_found]
- [readme] Filtering invocation with rarefaction depth and prevalence filter: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] Consistency driver scripts: "run_obesity_*.sh --> runs the obesity datasets for the consistency across datasets analysis"
- [discussion] Obesity consistency null result: "the consistency of significant genera across five datasets comparing obese and control individuals was no higher than expected by chance for most tools."
