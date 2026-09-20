---
name: roc-curve-computation-differential-abundance-asvs
description: Use when you have significant-ASV calls from multiple DA tools (e.g.,
  ANCOM-II, ALDEx2, LEfSe, edgeR, Wilcoxon) plus false-positive analysis tables, and
  need to score each tool's discriminatory power via AUROC, precision, recall, and
  F1.
when_to_use_negative:
- When the goal is to determine which taxa are significantly differentially abundant
  rather than to benchmark the discriminatory performance of tool calls
- When the data have not been pre-processed by the paper's pipeline (10% prevalence
  filtering, filtered/unfiltered dataset runs) — results depend on data pre-processing
- When only a single tool's calls exist; the skill's purpose is cross-tool comparison
  of AUROC, precision, and recall
tools:
- name: ROCR
  role: Compute per-ASV ROC curves and AUC values from DA-tool significance calls
    used as prediction scores
- name: dplyr
  role: Aggregate per-ASV AUCs into per-tool mean AUROC and metric summaries
- name: ggplot2
  role: Plot ROC curves and per-tool precision/recall/F1 summaries (Supplementary
    Figs. 3-4 layout)
- name: ALDEx2
  role: Source of significant-ASV calls evaluated for discrimination (highest precision
    after ANCOM-II at threshold 0.7)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:31af58914e77f066e2b94bc0a3498f182f78b210d488fe4c0e89959be889b441
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALDEx2
  - ROCR
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# roc-curve-computation-differential-abundance-asvs

## Summary

Computes per-ASV ROC curves, AUCs, and threshold-binarized precision/recall/F1 to evaluate how well significant ASVs called by each differential abundance (DA) tool discriminate between sample groups, run separately on relative-abundance and CLR-transformed inputs from the 38 filtered 16S rRNA datasets.

## When to use

Use when you have significant-ASV calls from multiple DA tools (e.g., ANCOM-II, ALDEx2, LEfSe, edgeR, Wilcoxon) plus false-positive analysis tables, and need to score each tool's discriminatory power via AUROC, precision, recall, and F1.

## When NOT to use

- When the goal is to determine which taxa are significantly differentially abundant rather than to benchmark the discriminatory performance of tool calls
- When the data have not been pre-processed by the paper's pipeline (10% prevalence filtering, filtered/unfiltered dataset runs) — results depend on data pre-processing
- When only a single tool's calls exist; the skill's purpose is cross-tool comparison of AUROC, precision, and recall

## Inputs

- Significant-ASV result tables from DA tools run on the 38 filtered datasets (run_0.1_filt_datasets.sh output)
- False-positive analysis tables from run_all_FP_datasets_filt.sh
- Sample-group labels from the two-column TSV metadata files (sample names + sample groupings)
- Relative-abundance and CLR-transformed ASV abundance matrices

## Outputs

- Summary TSV with mean AUROC, precision, recall, and F1 for every tool under both input types (relative abundance and CLR)
- Per-ASV ROC curves and AUC values computed with ROCR
- ggplot2 figures reproducing the Supplementary Figs. 3-4 layout

## How to apply

Load the significant-ASV result tables produced by the DA tools on the 38 filtered datasets (generated via run_0.1_filt_datasets.sh) together with the false-positive analysis tables from run_all_FP_datasets_filt.sh in R. For each tool, treat its significant-ASV calls as prediction scores and compute per-ASV ROC curves and AUC values with ROCR, running the computation separately on relative-abundance inputs and on CLR-transformed inputs. Aggregate the per-ASV AUCs into a mean AUROC per tool per input type. Binarize performance at AUC thresholds 0.7 and 0.9 and compute precision, recall, and F1 per tool; then plot ROC curves and per-tool metric summaries with ggplot2 (dplyr for aggregation) to reproduce the Supplementary Figs. 3-4 layout.

## Related tools

- **ROCR** (Compute per-ASV ROC curves and AUC values from DA-tool significance calls used as prediction scores)
- **dplyr** (Aggregate per-ASV AUCs into per-tool mean AUROC and metric summaries)
- **ggplot2** (Plot ROC curves and per-tool precision/recall/F1 summaries (Supplementary Figs. 3-4 layout))
- **ALDEx2** (Source of significant-ASV calls evaluated for discrimination (highest precision after ANCOM-II at threshold 0.7))

## Examples

```
library(ROCR); pred <- prediction(scores, labels); auc <- performance(pred, "auc")@y.values[[1]]
```

## Evaluation signals

- Summary TSV exists and contains an entry for every DA tool under both input types (relative abundance and CLR)
- Computed mean AUROC, precision, recall, and F1 match the reported Supplementary Fig. 3-4 values (e.g., ANCOM-II precision 0.99 and ALDEx2 precision 0.82 at threshold 0.7 on relative abundance; Wilcoxon on CLR reaches F1 of 0.70)
- Median precision for ANCOM-II ~0.99 and ALDEx2 ~0.82 on relative abundance (and 1.0 / 0.83 on CLR) at AUROC threshold 0.7
- ANCOM-II and ALDEx2 show lower recall than high-recall tools like LEfSe and edgeR, as reported
- Each ROC computation was run separately for relative-abundance and CLR inputs, yielding two AUROC values per tool

## Limitations

- Recall/precision depend on the binarization threshold (0.7 vs 0.9 AUROC), so tool rankings can shift between thresholds
- The false-positive analysis relies on randomized groupings, which 'is not a perfect representation of real data'; few significant ASVs under this approach is not proof of low practical FDR
- Discriminatory performance reflects the paper's specific pre-processing (10% prevalence filter, filtered vs unfiltered datasets) and may not transfer to other pipelines

## Evidence

- [results] At an AUROC threshold of 0.7, ANCOM-II and ALDEx2 had the highest precision on both relative abundance (medians 0.99 and 0.82): [evidence span withheld — anchor status: not_found]
- [methods] compute per-ASV ROC curves and AUC values using ROCR: [evidence span withheld — anchor status: not_found]
- [methods] Binarize tool performance at AUC thresholds 0.7 and 0.9 and compute precision, recall, and F1 for every tool: [evidence span withheld — anchor status: not_found]
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [abstract] these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
