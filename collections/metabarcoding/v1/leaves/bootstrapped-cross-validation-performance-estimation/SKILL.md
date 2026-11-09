---
name: bootstrapped-cross-validation-performance-estimation
description: Use when comparing model performance between two feature sets (e.g.,
  MetaPhlAn 4 vs MetaPhlAn 3 taxonomic profiles) against a panel of markers and you
  need a robust, bootstrap-replicated estimate of AUC/R2 deltas.
when_to_use_negative:
- Input is already a single fitted model or a held-out test score and no comparison
  across feature sets is required — simple cross-validation suffices.
- The task is taxonomic profiling itself (generating SGB abundance tables from reads)
  rather than downstream model evaluation — use MetaPhlAn profiling, not this evaluation
  skill.
- Sample size is too small for repeated 80/20 splits to yield stable per-split AUC
  estimates (the anchor cohort here is n=1,001).
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MetaPhlAn 4
  role: Generates the SGB relative abundance feature matrices (MetaPhlAn 4 and MetaPhlAn
    3 variants) whose predictive performance is compared via bootstrapped cross-validation.
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Baseline profiling tool providing the MetaPhlAn 3-based SGB profiles used
    as the comparison feature set.
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:f8e7431c3dc9683ad23b609558821ee623681ab0ab54bc1d0388ab07777dca16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Bootstrapped cross-validation performance estimation

## Summary

Estimates and compares predictive performance of random forest classifiers/regressors (e.g., on MetaPhlAn 4 vs MetaPhlAn 3 SGB abundance profiles) using 80/20 train–test splits repeated over 100 bootstrap iterations, reporting cross-validated AUC/R2 medians and per-marker deltas.

## When to use

Use when comparing model performance between two feature sets (e.g., MetaPhlAn 4 vs MetaPhlAn 3 taxonomic profiles) against a panel of markers and you need a robust, bootstrap-replicated estimate of AUC/R2 deltas.

## When NOT to use

- Input is already a single fitted model or a held-out test score and no comparison across feature sets is required — simple cross-validation suffices.
- The task is taxonomic profiling itself (generating SGB abundance tables from reads) rather than downstream model evaluation — use MetaPhlAn profiling, not this evaluation skill.
- Sample size is too small for repeated 80/20 splits to yield stable per-split AUC estimates (the anchor cohort here is n=1,001).

## Inputs

- MetaPhlAn 4 SGB relative abundance matrix (e.g., 1,001 ZOE PREDICT 1 individuals x SGB features)
- MetaPhlAn 3-based SGB relative abundance matrix for the same samples
- Panel of 19 nutritional and cardiometabolic marker measurements (e.g., ASCVD 10-year risk, Healthy Eating Index, aMED, uPDI)

## Outputs

- Per-marker median cross-validated AUC (classifiers) or R2 (regressors) over 100 bootstrap splits
- Per-marker delta AUC and percent improvement (MetaPhlAn 4 vs MetaPhlAn 3)
- Aggregated results table (marker_prediction_results.csv)

## How to apply

For each of the 19 nutritional/cardiometabolic markers, train a random forest classifier (binary/discrete markers) or regressor (continuous markers) on the SGB relative abundance matrix using 80/20 train–test splits repeated over 100 bootstrap iterations, recording cross-validated AUC or R2 per split. Repeat the identical procedure on the alternative feature set (MetaPhlAn 3-based SGB profiles) so the only difference between runs is the profiling version. Summarize performance per marker as the median AUC/R2 across the 100 splits, then compute the delta AUC and percent improvement per marker (ZOE PREDICT 1 anchors: median AUC 0.74 overall, +4.84% over MetaPhlAn 3; ASCVD risk +0.106 AUC / 16.24%; Healthy Eating Index +0.072 AUC / 10.05%). Assemble per-marker medians and deltas into a results table (marker_prediction_results.csv) so feature-set comparisons are auditable. Use MetaPhlAn 4 SGB profiles (covering 26,970 species-level genome bins) as the feature matrix, since uncharacterized taxa are strong host-condition biomarkers.

## Related tools

- **MetaPhlAn 4** (Generates the SGB relative abundance feature matrices (MetaPhlAn 4 and MetaPhlAn 3 variants) whose predictive performance is compared via bootstrapped cross-validation.) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Baseline profiling tool providing the MetaPhlAn 3-based SGB profiles used as the comparison feature set.) — https://github.com/biobakery/MetaPhlAn

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import train_test_split; X_tr,X_te,y_tr,y_te=train_test_split(X,y,test_size=0.2); auc=RandomForestClassifier().fit(X_tr,y_tr).score(X_te,y_te)
```

## Evaluation signals

- The overall median AUC should match the anchor value of 0.74 (a 4.84% improvement over the MetaPhlAn 3 baseline).
- ASCVD and Healthy Eating Index deltas should reproduce +0.106 and +0.072 AUC respectively (16.24% and 10.05% improvement).
- MetaPhlAn 4 features should improve prediction for 17 of the 19 markers versus MetaPhlAn 3.
- Each marker's metric must be the median over exactly 100 bootstrap-iterated 80/20 train–test splits (AUC for classifiers, R2 for regressors).
- marker_prediction_results.csv must contain per-marker median AUCs, MetaPhlAn 3 comparison deltas, and uSGB–marker partial Spearman correlations (e.g., aMED ρ=0.21, HEI ρ=0.19, uPDI ρ=-0.25).

## Limitations

- Many related downstream statistical challenges remain to be addressed; for example, the tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa is an unaddressed statistical challenge.
- Results are anchored to a single cohort (ZOE PREDICT 1, n=1,001); deltas may not transfer to other cohorts or environments.
- Bootstrap-split medians are sensitive to the chosen split ratio (80/20) and iteration count (100); deviating from these breaks comparability with the anchor values.

## Evidence

- [methods] Train random forest classifiers/regressors for each of the 19 markers using 80/20 train–test splits repeated over 100 bootstrap iterations, recording the cross-validated AUC/R2 per split.: [evidence span withheld — anchor status: not_found]
- [results] reaching a new median AUC of 0.74 (4.84% improvement): [evidence span withheld — anchor status: not_found]
- [abstract] proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [abstract] previously undetected species are strong biomarkers for host conditions: [evidence span withheld — anchor status: not_found]
