---
name: hypervariable-region-comparison
description: Use when comparing paired 16S rRNA hypervariable region combinations
  (V1–V2, V3–V4, V5–V7, V7–V9) to decide which discriminates bacterial genera in sputum/respiratory
  samples, or to assign discriminant genera per region.
when_to_use_negative:
- Analysis of full-length 16S rRNA sequences from third-generation platforms (PacBio/Nanopore)
  — this skill addresses Illumina MiSeq's 300 bp limit by comparing sub-regions instead
- Non-16S amplicon data (e.g., ITS/fungal profiling), since the Greengenes reference
  and region design are bacterial 16S-specific
- Data already collapsed to a single fixed region (e.g., V4-only studies) with no
  multi-region comparison possible
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: LEfSe
  role: Identifies genera with significantly different relative abundances among region
    groups via log10-transformed LDA scores (threshold > 2, 30-fold bootstrap)
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database for taxonomic assignment, collapsed at genus level, used
    to cross-validate ZymoBIOMICS against samples
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Deblur
  role: Denoising pipeline producing bacterial amplicon sequence variants (ASVs) at
    the genus level for each region combination
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: FastQC
  role: Quality control of raw reads (Q30 sequences) prior to region comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:170c2dce977a54f7c0619519f57afb73aee50ecfebe5dc8814a1298f896d4563
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - LEfSe
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# hypervariable-region-comparison

## Summary

A comparative evaluation of 16S rRNA combined hypervariable regions (V1–V2, V3–V4, V5–V7, V7–V9) for their ability to discriminate bacterial genera in sputum samples, using LEfSe (LDA threshold > 2) on Greengenes-collapsed genus-level ASV tables. It determines which region combination — here V1–V2 — has the highest resolving power for taxonomic identification.

## When to use

Use when comparing paired 16S rRNA hypervariable region combinations (V1–V2, V3–V4, V5–V7, V7–V9) to decide which best discriminates bacterial genera in sputum/respiratory samples, or to assign discriminant genera per region.

## When NOT to use

- Analysis of full-length 16S rRNA sequences from third-generation platforms (PacBio/Nanopore) — this skill addresses Illumina MiSeq's 300 bp limit by comparing sub-regions instead
- Non-16S amplicon data (e.g., ITS/fungal profiling), since the Greengenes reference and region design are bacterial 16S-specific
- Data already collapsed to a single fixed region (e.g., V4-only studies) with no multi-region comparison possible

## Inputs

- Greengenes-collapsed genus-level ASV abundance table grouped by combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) from 33 human sputum samples
- Deblur-identified ASV sequences (genus level) from QIAseq 16S screening panel libraries sequenced on Illumina
- ZymoBIOMICS mock community control abundance data
- Standard-of-care culture results for cross-checking

## Outputs

- Region-by-genera LEfSe discriminant assignment table with LDA scores (threshold > 2)
- Ranked discriminant genera per hypervariable region combination
- ROC curve / AUC comparison of classifier accuracy per region
- Pairwise sensitivity and specificity comparison across regions
- Kruskal–Wallis alpha diversity and NMDS Bray–Curtis ordination summaries per region

## How to apply

Load the Greengenes-collapsed genus-level ASV abundance table from the authors' GitHub repository, grouped by the four combined hypervariable regions (V1–V2, V3–V4, V5–V7, V7–V9). Run LEfSe to identify genera with significantly different relative abundances among region groups, using an LDA threshold > 2 after scaling relative abundances to (1, 10exp9) and computing log10-transformed LDA scores, supported by the default 30-fold bootstrapping and averaging. Rank the discriminant genera per region by LDA score to build the region-by-genera assignment table (e.g., V1–V2 uniquely recovers Pseudomonas, plus Glesbergeria, Sinobaca, Ochromonas; V3–V4 recovers Prevotella, Corynebacterium, Filifactor). Cross-check recovered genera against standard-of-care culture results and the ZymoBIOMICS mock community control. Complement LEfSe with ROC-curve cross-validation of the microbiota classifier, pairwise sensitivity/specificity comparisons, and Kruskal–Wallis alpha diversity testing to rank region resolving power.

## Related tools

- **LEfSe** (Identifies genera with significantly different relative abundances among region groups via log10-transformed LDA scores (threshold > 2, 30-fold bootstrap)) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database for taxonomic assignment, collapsed at genus level, used to cross-validate ZymoBIOMICS against samples) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Deblur** (Denoising pipeline producing bacterial amplicon sequence variants (ASVs) at the genus level for each region combination) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **FastQC** (Quality control of raw reads (Q30 sequences) prior to region comparison)

## Examples

```
run_lefse.py --lda 2.0 --nlogs 9 --bootstrap 30 --output region_genera_lefse.txt greengenes_genus_table_by_region.txt
```

## Evaluation signals

- Each discriminant genus has a log10-transformed LDA score > 2 in the region group(s) where it is assigned
- The region-by-genera table reproduces distinct genus sets per region (e.g., Pseudomonas recovered only by V1–V2)
- ROC AUC is highest for V1–V2, consistent with the reported highest resolving power
- Sensitivity/specificity against the ZymoBIOMICS mock community is maximal for V1–V2, with highest sample–mock concordance in that region
- Kruskal–Wallis tests show significant alpha diversity differences between region groups (V1–V2, V3–V4, V5–V7 higher than V7–V9)

## Limitations

- Illumina MiSeq 300 bp read length limits 16S rRNA profiling and constrains which region combinations can be recovered
- Regions were trained with Greengenes despite the availability of more updated versions of SILVA, risking reference database mismatch
- The ZymoBIOMICS mock community contains only 10 species, which affects sensitivity and specificity of taxonomic classification
- No comparison to full-length 16S rRNA sequencing was performed to validate the region rankings

## Evidence

- [abstract] The combined hypervariable regions showed significant differences when discriminating between the relative abundances of bacterial genera.: "The combined hypervariable regions showed significant differences when discriminating between the relative abundances of bacterial genera."
- [abstract] The area under the curve revealed that V1–V2 had the highest resolving power for accurately identifying respiratory bacterial taxa from sputum samples.: "The area under the curve revealed that V1–V2 had the highest resolving power for accurately identifying respiratory bacterial taxa from sputum samples."
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [results] We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS): "We used a receiver operating characteristic (ROC) curve for cross-validation accuracy of the microbiota classifier in the microbial standard control (ZymoBIOMICS)"
- [discussion] despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes: "despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes"
