---
name: auc-confidence-interval-estimation
description: Use when cross-validating taxonomic classifiers (e.g., Deblur/Greengenes
  genus-level ASV tables from different 16S hypervariable regions) against a labeled
  mock-community control and you need AUC with a 0.95 confidence interval to rank
  region performance.
when_to_use_negative:
- Input is raw reads rather than a denoised genus-level ASV table — the regions must
  first be processed with Deblur and Greengenes-based assignment collapsed at genus
  level
- There is no labeled positive control (e.g., ZymoBIOMICS mock community) to define
  reference taxa, so ROC sensitivity/specificity cannot be cross-validated
- Sequencing is full-length third-generation (e.g., Nanopore) 16S rRNA, where hypervariable-region
  choice is not the discriminating variable
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: Deblur
  role: Denoises reads into bacterial ASVs at the genus level, providing the per-region
    abundance tables on which ROC/AUC is computed
- name: Greengenes
  role: Reference database used as the microbiota classifier for cross-validation
    between ZymoBIOMICS and respiratory samples
- name: R (version 3.4.1)
  role: Statistical environment in which diversity and ROC/AUC analyses were performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:b9d040e04bd321ac1f94ff32bfdf645af92033bf9f0e05d38b5f6b2b8662e49e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Greengenes
  - R (version 3.4.1)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# auc-confidence-interval-estimation

## Summary

Estimates ROC curves and area-under-the-curve (AUC) values with 95% confidence intervals to quantify which 16S rRNA hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) best classifies respiratory taxa against a ZymoBIOMICS mock-community positive control.

## When to use

Use when cross-validating taxonomic classifiers (e.g., Deblur/Greengenes genus-level ASV tables from different 16S hypervariable regions) against a labeled mock-community control and you need AUC with a 0.95 confidence interval to rank region performance.

## When NOT to use

- Input is raw reads rather than a denoised genus-level ASV table — the regions must first be processed with Deblur and Greengenes-based assignment collapsed at genus level
- There is no labeled positive control (e.g., ZymoBIOMICS mock community) to define reference taxa, so ROC sensitivity/specificity cannot be cross-validated
- Sequencing is full-length third-generation (e.g., Nanopore) 16S rRNA, where hypervariable-region choice is not the discriminating variable

## Inputs

- Genus-level ASV abundance tables per hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) from the Deblur pipeline with Greengenes taxonomic assignment
- ZymoBIOMICS microbial community standard composition used as positive-control labels

## Outputs

- Pair-wise ROC curves per combined hypervariable region
- AUC values with 0.95 confidence intervals (IQR) per region, assembled as roc_auc_table.csv

## How to apply

Load the genus-level ASV abundance tables for each combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9), produced upstream by Deblur denoising with Greengenes-based taxonomic assignment collapsed at genus level. Use the ZymoBIOMICS microbial community standard taxa as the reference (positive-control) labels for cross-validation, as done in R (version 3.4.1). For each region, compute a receiver operating characteristic (ROC) curve assessing sensitivity and specificity of the microbiota classifier against the ZymoBIOMICS control, with a 0.95 confidence interval on the AUC. Report the AUC as a median with an IQR interval (e.g., V1–V2: 0.736, IQR 0.566–0.906) rather than a point estimate alone, so overlapping regions (V3–V4 0.474, V5–V7 0.462, V7–V9 0.581) can be judged non-significant. Rank regions by AUC; only intervals excluding ~0.5 are treated as having significant resolving power.

## Related tools

- **Deblur** (Denoises reads into bacterial ASVs at the genus level, providing the per-region abundance tables on which ROC/AUC is computed)
- **Greengenes** (Reference database used as the microbiota classifier for cross-validation between ZymoBIOMICS and respiratory samples)
- **R (version 3.4.1)** (Statistical environment in which diversity and ROC/AUC analyses were performed)

## Evaluation signals

- roc_auc_table.csv contains exactly four region rows (V1–V2, V3–V4, V5–V7, V7–V9) with all AUC values in (0, 1)
- V1–V2 attains the highest AUC, reported as 0.736 with IQR 0.566–0.906, ranking above V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581)
- Only V1–V2's interval excludes ~0.5 (the chance-classification AUC); the other three regions fall near or below 0.5
- Each region's AUC is accompanied by a 0.95 confidence interval, not a bare point estimate
- ROC computation uses ZymoBIOMICS taxa as reference labels, matching the mock-community composition

## Limitations

- The mock microbial community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification
- Regions were trained with the outdated Greengenes database instead of more updated SILVA versions
- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling, constraining which regions can be assessed

## Evidence

- [results] ROC cross-validation design: "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [results] Pairwise per-region comparison: "Then, we performed pair-wise comparisons with each combined hypervariable region (V1–V2, V3–V4, V5–V7, and V7–V9) to compare sensitivity and specificity"
- [abstract] V1–V2 highest AUC outcome: "The area under the curve revealed that V1–V2 had the highest resolving power for accurately identifying respiratory bacterial taxa from sputum samples."
- [methods] Positive-control input: "A ZymoBIOMICS microbial community standard control was used as a positive control"
- [discussion] 10-species mock community caveat: "mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification"
