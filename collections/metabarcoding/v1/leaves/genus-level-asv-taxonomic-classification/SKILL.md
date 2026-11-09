---
name: genus-level-asv-taxonomic-classification
description: Use when raw 16S rRNA FASTQ data from sputum or mock-community samples
  must be denoised to ASVs, assigned taxonomy at genus level, and cross-validated
  for sensitivity/specificity against a ZymoBIOMICS standard control.
when_to_use_negative:
- The goal is full-length 16S rRNA taxonomic resolution — the study explicitly does
  not compare against full-length third-generation sequencing and recommends it as
  future work.
- Species-level or strain-level identification is required — taxonomic assignments
  here are collapsed at the genus level only.
- Estimating absolute sensitivity/specificity on complex communities — the ZymoBIOMICS
  mock control contains only 10 species, which limits sensitivity/specificity estimates.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: Deblur
  role: Denoises paired-end reads into non-chimeric bacterial amplicon sequence variants
    (ASVs) used for genus-level classification
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database used to obtain taxonomic assignments for ASVs, collapsed
    at the genus level, and as the microbiota classifier for cross-validation against
    ZymoBIOMICS
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: QIIME 2
  role: Provides the non-chimeric 16S rRNA database used to identify chimeras in the
    reads before ASV denoising
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: FastQC
  role: Assesses raw-read quality (Q30 sequences) and filtering with a cut-off score
    of up to 3011 prior to denoising
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:dbbed90622e2771eeef725a11202c86d7c58f5ffd742cd58d3fef9fd4900ad73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genus-level-asv-taxonomic-classification

## Summary

Classifies 16S rRNA amplicons from respiratory (sputum) samples into bacterial ASVs collapsed at genus level using Deblur denoising and Greengenes taxonomy, then benchmarks classification accuracy against the ZymoBIOMICS mock community via ROC/AUC to identify the best-performing hypervariable region.

## When to use

Use when raw 16S rRNA FASTQ data from sputum or mock-community samples must be denoised to ASVs, assigned taxonomy at genus level, and cross-validated for sensitivity/specificity against a ZymoBIOMICS standard control.

## When NOT to use

- The goal is full-length 16S rRNA taxonomic resolution — the study explicitly does not compare against full-length third-generation sequencing and recommends it as future work.
- Species-level or strain-level identification is required — taxonomic assignments here are collapsed at the genus level only.
- Estimating absolute sensitivity/specificity on complex communities — the ZymoBIOMICS mock control contains only 10 species, which limits sensitivity/specificity estimates.

## Inputs

- Paired-end 16S rRNA Illumina FASTQ reads from sputum samples (library made with QIAseq 16S/ITS screening panel)
- Raw reads of the ZymoBIOMICS mock community standard control
- Greengenes reference database for taxonomic assignment
- QIIME 2 non-chimeric 16S rRNA reference database for chimera checking

## Outputs

- Genus-level ASV abundance tables per hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)
- ROC/AUC comparison table with confidence intervals per region (roc_auc_table.csv)
- Ranked assessment of sensitivity and specificity of each region against the ZymoBIOMICS control

## How to apply

Isolate DNA (33 sputum samples plus a ZymoBIOMICS microbial community standard positive control), prepare libraries with the QIAseq 16S/ITS screening panel for Illumina, and check raw reads with FastQC (Q30 quality, cut-off score up to 3011). Denoise paired-end reads with the Deblur pipeline in QIIME 2, first removing chimeras flagged against the non-chimeric 16S rRNA database at a best hit similarity rate of less than 99%, then assign taxonomy with the Greengenes database and collapse at the genus level. For validation, use the ZymoBIOMICS taxa as positive-control labels and run pair-wise ROC curve analysis per combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9), reporting AUC with intervals; the V1–V2 region is expected to attain the highest AUC (0.736, IQR 0.566–0.906) and the highest concordance with the mock control.

## Related tools

- **Deblur** (Denoises paired-end reads into non-chimeric bacterial amplicon sequence variants (ASVs) used for genus-level classification) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database used to obtain taxonomic assignments for ASVs, collapsed at the genus level, and as the microbiota classifier for cross-validation against ZymoBIOMICS) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **QIIME 2** (Provides the non-chimeric 16S rRNA database used to identify chimeras in the reads before ASV denoising) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **FastQC** (Assesses raw-read quality (Q30 sequences) and filtering with a cut-off score of up to 3011 prior to denoising) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Evaluation signals

- roc_auc_table.csv contains exactly four region rows (V1–V2, V3–V4, V5–V7, V7–V9), each with an AUC value in (0,1) and a reported interval.
- V1–V2 attains the highest AUC (reported 0.736, IQR 0.566–0.906), ranking above V3–V4 (0.474), V5–V7 (0.462), and V7–V9 (0.581).
- Taxonomic assignments are collapsed at the genus level using Greengenes; no species- or strain-level labels should appear in the ASV tables.
- Chimeric reads are removed before denoising, with a best hit similarity rate of less than 99% against the QIIME 2 non-chimeric 16S rRNA database.
- Raw reads passing FastQC quality control show Q30 scores before downstream processing.

## Limitations

- The ZymoBIOMICS mock microbial community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification.
- The different regions were trained with Greengenes despite the availability of more updated versions of SILVA.
- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling; full-length comparison was not performed.
- Deblur is recommended for shorter bp fragments, which constrains longer-region combinations.

## Evidence

- [abstract] We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [methods] The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [results] We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS): "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [discussion] The highest concordance between samples and the ZymoBIOMICS microbial community control was also found in the V1–V2 region: "The highest concordance between samples and the ZymoBIOMICS microbial community control was also found in the V1–V2 region"
- [discussion] mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification: "mock microbial communities there are only 10 species and this fact affects in the sensitivity and specificity related with taxonomic classification"
