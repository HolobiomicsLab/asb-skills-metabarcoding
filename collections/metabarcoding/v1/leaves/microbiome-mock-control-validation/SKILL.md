---
name: microbiome-mock-control-validation
description: Use when amplicon ASV abundance tables (e.g., Deblur-derived, genus-level)
  must be cross-validated against a mock community standard such as ZymoBIOMICS to
  quantify classifier sensitivity/specificity via ROC/AUC.
when_to_use_negative:
- The dataset has no positive-control mock community (e.g., no ZymoBIOMICS standard
  sequenced alongside samples), so cross-validation labels cannot be defined.
- Sequences come from third-generation full-length 16S rRNA platforms (e.g., Nanopore)
  rather than the Illumina MiSeq 300 bp-limited short-amplicon setting the region-pair
  comparison addresses.
- Input is already a validated classifier output or ROC table; no re-computation of
  region-level AUCs is needed.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: Deblur
  role: Denoises Illumina amplicon reads into bacterial ASVs at the genus level, producing
    the region-specific abundance tables validated against the mock control
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database used for taxonomic assignment (collapsed at genus level)
    and as the microbiota classifier cross-validated against ZymoBIOMICS
- name: FastQC
  role: Quality assessment (Q30 sequences) and read filtering of raw data prior to
    denoising and mock-control validation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:7f645e9a1708cbc6074d8b9f0dacdb624ed3fee06355272770e37bfcf62d05f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - Deblur
  - FastQC
  - Greengenes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# microbiome-mock-control-validation

## Summary

Validates a 16S rRNA microbiota classifier against a positive-control mock community by computing pair-wise ROC curves and AUC per hypervariable region, as done with the ZymoBIOMICS standard to determine which region pair has the highest sensitivity and specificity for respiratory taxa.

## When to use

Use when amplicon ASV abundance tables (e.g., Deblur-derived, genus-level) must be cross-validated against a mock community standard such as ZymoBIOMICS to quantify classifier sensitivity/specificity via ROC/AUC.

## When NOT to use

- The dataset has no positive-control mock community (e.g., no ZymoBIOMICS standard sequenced alongside samples), so cross-validation labels cannot be defined.
- Sequences come from third-generation full-length 16S rRNA platforms (e.g., Nanopore) rather than the Illumina MiSeq 300 bp-limited short-amplicon setting the region-pair comparison addresses.
- Input is already a validated classifier output or ROC table; no re-computation of region-level AUCs is needed.

## Inputs

- Genus-level ASV abundance tables per hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) from Deblur denoising with Greengenes taxonomy collapsed at genus level
- ZymoBIOMICS microbial community standard composition used as positive-control reference labels

## Outputs

- ROC curves per combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)
- roc_auc_table.csv: AUC values with confidence intervals per region (four rows)

## How to apply

Load genus-level ASV abundance tables for each combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) produced by the Deblur pipeline with Greengenes-based taxonomic assignment collapsed at genus level. Use the ZymoBIOMICS microbial community standard composition as the positive-control reference labels. Perform pair-wise ROC curve analysis for each region, assessing sensitivity and specificity of the classifier against the mock control, and compute AUC with confidence intervals. Assemble the four region AUC values into a table (roc_auc_table.csv). Rationale: the article's ROC cross-validation showed V1–V2 achieved the only significant AUC (0.736, IQR 0.566–0.906) versus V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581), identifying V1–V2 as the most sensitive and specific region pair.

## Related tools

- **Deblur** (Denoises Illumina amplicon reads into bacterial ASVs at the genus level, producing the region-specific abundance tables validated against the mock control) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database used for taxonomic assignment (collapsed at genus level) and as the microbiota classifier cross-validated against ZymoBIOMICS)
- **FastQC** (Quality assessment (Q30 sequences) and read filtering of raw data prior to denoising and mock-control validation)

## Evaluation signals

- roc_auc_table.csv contains exactly four region rows (V1–V2, V3–V4, V5–V7, V7–V9) with AUC values strictly within (0,1).
- V1–V2 attains the highest AUC, reported as 0.736 (IQR 0.566–0.906), ranking above V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581).
- AUC confidence intervals (0.95) are present for every region row.
- Only V1–V2's AUC is significant; the other regions should show non-significant AUCs consistent with the published ranking.
- Taxonomic labels derive from the ZymoBIOMICS standard taxa used as positive-control references, not from patient sample labels.

## Limitations

- The ZymoBIOMICS mock community contains only 10 species, which affects the achievable sensitivity and specificity of taxonomic classification.
- Regions were trained with the outdated Greengenes database instead of more updated SILVA versions.
- The Illumina MiSeq 300 bp read-length limit constrains 16S rRNA profiling of the longer region combinations.
- No comparison to full-length 16S rRNA sequencing was performed, so absolute (rather than relative) accuracy of region pairs is untested.

## Evidence

- [results] ROC cross-validation of region classifiers: "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [results] Pair-wise region comparisons: "Then, we performed pair-wise comparisons with each combined hypervariable region (V1–V2, V3–V4, V5–V7, and V7–V9) to compare sensitivity and specificity"
- [methods] Mock community as positive control: "A ZymoBIOMICS microbial community standard control was used as a positive control"
- [methods] Genus-level taxonomy assignment: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [discussion] Mock community size limitation: "mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification"
