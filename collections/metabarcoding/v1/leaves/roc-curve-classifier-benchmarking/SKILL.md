---
name: roc-curve-classifier-benchmarking
description: Use when genus-level ASV abundance tables from multiple 16S rRNA hypervariable
  regions need to be ranked by taxonomic-classification accuracy against a positive-control
  mock community.
when_to_use_negative:
- Input data are not genus-level 16S ASV abundances (e.g. metagenomic functional profiles)
  — the mock-community cross-validation is grounded in 16S rRNA region taxonomy.
- No positive-control/mock community sample exists in the dataset — ROC cross-validation
  against ZymoBIOMICS reference taxa cannot be performed.
- Regions were sequenced with long-read full-length platforms — the MiSeq 300 bp limit
  that motivated this region comparison does not apply.
tools:
- name: Deblur
  role: Denoises Illumina reads to identify bacterial ASVs at genus level, producing
    the abundance tables that serve as classifier features for ROC analysis
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database used for taxonomic assignment (collapsed at genus level)
    and as the microbiota classifier cross-validated against the ZymoBIOMICS control
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:1acb02b5bf90ddf342ac9aaebaa143385490f10788f96778251b7d121ccc8a5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Greengenes
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# roc-curve-classifier-benchmarking

## Summary

Benchmark 16S rRNA hypervariable-region taxonomic classifiers by pairwise ROC curve analysis against the ZymoBIOMICS mock community standard, comparing sensitivity, specificity, and area under the curve (AUC) for each combined region (V1–V2, V3–V4, V5–V7, V7–V9).

## When to use

Use when genus-level ASV abundance tables from multiple 16S rRNA hypervariable regions need to be ranked by taxonomic-classification accuracy against a positive-control mock community.

## When NOT to use

- Input data are not genus-level 16S ASV abundances (e.g. metagenomic functional profiles) — the mock-community cross-validation is grounded in 16S rRNA region taxonomy.
- No positive-control/mock community sample exists in the dataset — ROC cross-validation against ZymoBIOMICS reference taxa cannot be performed.
- Regions were sequenced with long-read full-length platforms — the MiSeq 300 bp limit that motivated this region comparison does not apply.

## Inputs

- Genus-level ASV abundance tables per hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) from Deblur/Greengenes processing
- ZymoBIOMICS mock community standard taxonomic composition (positive-control labels)

## Outputs

- ROC curves per combined hypervariable region
- AUC table with confidence/IQR intervals per region (e.g. roc_auc_table.csv)
- Ranked comparison of sensitivity and specificity across regions

## How to apply

Load the genus-level ASV abundance tables produced by the Deblur denoising pipeline, with taxonomy assigned via the Greengenes database and collapsed at genus level. Retrieve the ZymoBIOMICS microbial community standard composition and use its taxa as the positive-control reference labels. Perform pair-wise ROC curve comparisons of each combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) against the ZymoBIOMICS control to assess sensitivity and specificity, reporting AUC with confidence/IQR intervals. In the source study this identified V1–V2 as the highest-resolving region (AUC 0.736, IQR 0.566–0.906), exceeding V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581).

## Related tools

- **Deblur** (Denoises Illumina reads to identify bacterial ASVs at genus level, producing the abundance tables that serve as classifier features for ROC analysis) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database used for taxonomic assignment (collapsed at genus level) and as the microbiota classifier cross-validated against the ZymoBIOMICS control) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Evaluation signals

- Output table contains exactly four region rows (V1–V2, V3–V4, V5–V7, V7–V9) with AUC values in (0,1).
- V1–V2 attains the highest AUC (expected 0.736, IQR 0.566–0.906), ranking above V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581).
- Each AUC is reported with a confidence interval (e.g. IQR) rather than a point estimate alone.
- Reference labels trace back to the ZymoBIOMICS mock community standard, not to sample-derived annotations.

## Limitations

- The mock microbial community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification.
- Regions were trained with the outdated Greengenes database despite more updated SILVA versions being available.
- The Illumina MiSeq 300 bp length limit constrains 16S rRNA profiling and may bias region-specific performance.
- No comparison to full-length 16S rRNA sequencing was performed, so absolute (not just relative) region accuracy is unvalidated.

## Evidence

- [results] ROC cross-validation against ZymoBIOMICS: "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [results] Pairwise region comparisons: "Then, we performed pair-wise comparisons with each combined hypervariable region (V1–V2, V3–V4, V5–V7, and V7–V9) to compare sensitivity and specificity"
- [methods] Taxonomic preprocessing: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [abstract] Expected outcome: "The area under the curve revealed that V1–V2 had the highest resolving power for accurately identifying respiratory bacterial taxa from sputum samples."
