---
name: mock-community-taxa-comparison
description: Use when amplicon ASV tables from one or more 16S hypervariable regions
  are available alongside a ZymoBIOMICS mock community control, and the goal is to
  quantify which region(s) classify taxa with the sensitivity/specificity.
when_to_use_negative:
- Samples lack a mock community standard control, so no reference taxa exist for ROC
  cross-validation
- The question is compositional profiling (e.g. alpha diversity or NMDS of Bray–Curtis
  dissimilarities) rather than classification accuracy against a positive control
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: Deblur
  role: Denoises amplicon reads into ASVs collapsed at genus level, providing the
    abundance tables fed into the ROC comparison
  repo: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database used for taxonomic assignment and as the microbiota classifier
    cross-validated against ZymoBIOMICS
- name: FastQC
  role: Assessed Q30 quality and filtering of raw reads upstream of the ROC analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:2fb01866ab203557dc846d01543eb04fd60ae3486ccd255a41c1d108dc2cc475
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - Deblur
  - FastQC
  - Greengenes
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mock-community-taxa-comparison

## Summary

Cross-validates the accuracy of 16S rRNA hypervariable-region taxonomic classification by comparing mock community (ZymoBIOMICS) standard control taxa against sample taxa, using ROC/AUC to rank region combinations by sensitivity and specificity.

## When to use

Use when amplicon ASV tables from one or more 16S hypervariable regions are available alongside a ZymoBIOMICS mock community control, and the goal is to quantify which region(s) classify taxa with the best sensitivity/specificity.

## When NOT to use

- Samples lack a mock community standard control, so no reference taxa exist for ROC cross-validation
- The question is compositional profiling (e.g. alpha diversity or NMDS of Bray–Curtis dissimilarities) rather than classification accuracy against a positive control

## Inputs

- Genus-level ASV abundance tables per hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) from Deblur with Greengenes taxonomy
- ZymoBIOMICS mock community (microbial standard control) composition as reference taxa

## Outputs

- ROC curve per combined hypervariable region with confidence intervals
- AUC table (e.g. roc_auc_table.csv) listing AUC values and intervals per region
- Ranking of region combinations by sensitivity and specificity against the mock control

## How to apply

Take genus-level ASV abundance tables produced by the Deblur denoising pipeline with Greengenes-based taxonomy collapsed at genus level, for each combined region (V1–V2, V3–V4, V5–V7, V7–V9). Treat the ZymoBIOMICS microbial community standard's known taxa as positive-control reference labels and run a receiver operating characteristic (ROC) analysis for cross-validation accuracy of the microbiota classifier. Perform pair-wise comparisons of each combined hypervariable region to compare sensitivity and specificity, computing AUC with confidence intervals per region. Assemble a table of AUC values with intervals per region; here V1–V2 was the only significant region (AUC 0.736, IQR 0.566–0.906), exceeding V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581).

## Related tools

- **Deblur** (Denoises amplicon reads into ASVs collapsed at genus level, providing the abundance tables fed into the ROC comparison) — https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database used for taxonomic assignment and as the microbiota classifier cross-validated against ZymoBIOMICS)
- **FastQC** (Assessed Q30 quality and filtering of raw reads upstream of the ROC analysis)

## Evaluation signals

- The AUC table contains exactly four region rows (V1–V2, V3–V4, V5–V7, V7–V9) with AUC values in (0,1)
- V1–V2 attains the highest AUC (reported as 0.736, IQR 0.566–0.906), ranking above V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581)
- The only statistically significant AUC corresponds to V1–V2; other regions should not exceed it
- Genus-level taxonomy is confirmed to derive from Greengenes collapsed at genus level before comparison
- Mock community taxa used as reference labels match the ZymoBIOMICS standard control composition

## Limitations

- The mock microbial community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification
- Regions were trained with Greengenes despite more updated SILVA versions being available
- The 300 bp length limit of the Illumina MiSeq platform constrains 16S rRNA profiling for longer region combinations

## Evidence

- [results] ROC cross-validation method: "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [results] Pair-wise region comparisons: "Then, we performed pair-wise comparisons with each combined hypervariable region (V1–V2, V3–V4, V5–V7, and V7–V9) to compare sensitivity and specificity"
- [abstract] Highest sensitivity/specificity finding: "Comparing the taxa of microbial community standard control with the taxa samples, V1–V2 combination exhibits the most sensitivity and specificity."
- [discussion] Mock community size caveat: "mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification"
- [methods] Greengenes taxonomy assignment: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
