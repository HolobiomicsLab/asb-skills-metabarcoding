---
name: sputum-microbiome-culture-concordance-check
description: Use when genus-level 16S ASV assignments (e.g. Greengenes-collapsed sputum
  profiles) must be validated against a ZymoBIOMICS mock community standard and/or
  standard-of-care respiratory culture results before trusting region-specific taxa.
when_to_use_negative:
- Input is raw reads or non-collapsed ASV tables — taxonomy must first be assigned
  with Greengenes and collapsed at the genus level before concordance checking
- Samples lack a mock community control and standard-of-care culture results, leaving
  no reference against which to judge concordance
- The study goal is comparing full-length 16S rRNA sequencing or regions outside the
  four combined region combinations (e.g. V4–V6), which this article did not perform
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Greengenes
  role: Taxonomic classifier database used to assign genus-level labels and cross-validate
    between ZymoBIOMICS and sputum samples
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Deblur
  role: Identifies bacterial ASVs at the genus level that feed the abundance table
    being concordance-checked
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:2deb2f21c596d02d2fb96072e1ca54cbfd1fda49d4cc378fcb94ee07f9e88153
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - Deblur
  - Greengenes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sputum-microbiome-culture-concordance-check

## Summary

A validation skill for 16S rRNA taxonomic assignments of sputum samples: cross-check genus-level calls (and LEfSe discriminant genera) against the ZymoBIOMICS mock community control and standard-of-care culture results to verify which hypervariable region combination gives concordant taxa.

## When to use

Use when genus-level 16S ASV assignments (e.g. Greengenes-collapsed sputum profiles) must be validated against a ZymoBIOMICS mock community standard and/or standard-of-care respiratory culture results before trusting region-specific taxa.

## When NOT to use

- Input is raw reads or non-collapsed ASV tables — taxonomy must first be assigned with Greengenes and collapsed at the genus level before concordance checking
- Samples lack a mock community control and standard-of-care culture results, leaving no reference against which to judge concordance
- The study goal is comparing full-length 16S rRNA sequencing or regions outside the four combined region combinations (e.g. V4–V6), which this article did not perform

## Inputs

- Genus-level ASV abundance table collapsed with the Greengenes database (from FastQC Q30-filtered, Deblur-denoised, chimera-checked reads)
- ZymoBIOMICS Microbial Community standard control (mock community) profiles
- Standard-of-care culture results for the 33 sputum samples
- Per-region genus tables for the combined hypervariable regions (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- ROC-based cross-validation accuracy of the microbiota classifier against the mock community control
- Pair-wise sensitivity and specificity comparison of each combined hypervariable region
- Concordance assessment of LEfSe discriminant genera versus culture results and mock community taxa
- Region-by-genera assignment table (e.g. V1–V2 recovering Pseudomonas, Glesbergeria, Sinobaca, Ochromonas; V3–V4 recovering Prevotella, Corynebacterium, Filifactor)

## How to apply

Start from the Greengenes-collapsed genus-level ASV abundance table produced by the Deblur pipeline (FastQC Q30-filtered reads, non-chimeric reads identified against the QIIME 2 non-chimeric 16S rRNA database at <99% best-hit similarity). Assign taxonomy with the Greengenes database and collapse at the genus level, then cross-validate the classifier between the ZymoBIOMICS mock community control and sputum samples using a receiver operating characteristic (ROC) curve; the V1–V2 combination showed the most sensitivity and specificity versus the mock standard. Perform pair-wise comparisons of each combined region (V1–V2, V3–V4, V5–V7, V7–V9) to compare sensitivity and specificity, and cross-check LEfSe discriminant genera (LDA > 2, 30-fold bootstrapping) against standard-of-care culture results and the mock community. Judge success by concordance: mock-community taxa recovered with the highest concordance in the candidate region, and discriminant genera consistent with culture-confirmed pathogens.

## Related tools

- **Greengenes** (Taxonomic classifier database used to assign genus-level labels and cross-validate between ZymoBIOMICS and sputum samples) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Deblur** (Identifies bacterial ASVs at the genus level that feed the abundance table being concordance-checked) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Evaluation signals

- The candidate region combination recovers mock community taxa with the highest concordance and the most sensitivity and specificity versus the ZymoBIOMICS standard (V1–V2 in this study)
- ROC curve cross-validation of the Greengenes-based classifier between mock community and samples yields the highest area under the curve for the selected region (V1–V2)
- Each LEfSe discriminant genus is assigned only to region groups where it passes the LDA > 2 threshold, reproducing the reported pattern of distinct discriminant taxa per region
- Discriminant genera are consistent with standard-of-care culture results from the same sputum samples
- Reads upstream of the check passed FastQC Q30 filtering and chimera screening against the QIIME 2 non-chimeric 16S database (<99% best-hit similarity)

## Limitations

- The mock microbial community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification
- Illumina MiSeq's 300 bp length limit constrains 16S rRNA profiling of longer region combinations
- Regions were trained with Greengenes despite more updated versions of SILVA being available, which may bias taxonomic assignment

## Evidence

- [abstract] The analysis included a mock community as a microbial standard control (ZymoBIOMICS): "The analysis included a mock community as a microbial standard control (ZymoBIOMICS)"
- [results] We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS): "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [discussion] The highest concordance between samples and the ZymoBIOMICS microbial community control was also found in the V1–V2 region: "The highest concordance between samples and the ZymoBIOMICS microbial community control was also found in the V1–V2 region"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [discussion] mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification: "mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification"
