---
name: lefse-differential-abundance-analysis
description: Use when you have a genus-collapsed ASV abundance table grouped by experimental
  categories (e.g., 16S hypervariable region combinations) and need to identify taxa
  with significantly different relative abundances among groups.
when_to_use_negative:
- The abundance table is not collapsed at genus level against Greengenes (or an equivalent
  reference), since the reported discriminant taxa and LDA threshold behavior are
  genus-level.
- You need to compare taxonomic profiles against full-length third-generation 16S
  rRNA sequencing — the study explicitly did not perform this comparison.
- The sample categories are not discrete groups (e.g., region combinations, phenotype
  labels); LEfSe's LDA-based class discrimination requires group labels.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Greengenes
  role: Taxonomic classifier database used to obtain and collapse assignments at the
    genus level, producing the abundance table that LEfSe consumes
- name: Deblur
  role: Upstream denoising of paired-end 16S reads to identify bacterial ASVs at the
    genus level before LEfSe is run
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:e1c6aee6847ef79a50a5b00affb43d9af785c28be73d2e84a9f6ff8c32e747c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Greengenes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lefse-differential-abundance-analysis

## Summary

Runs LEfSe discriminative analysis on genus-level 16S rRNA ASV abundance tables (Greengenes-collapsed, from sputum samples) grouped by combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) to assign biomarker genera per region via LDA scores.

## When to use

Use when you have a genus-collapsed ASV abundance table grouped by experimental categories (e.g., 16S hypervariable region combinations) and need to identify taxa with significantly different relative abundances among groups.

## When NOT to use

- The abundance table is not collapsed at genus level against Greengenes (or an equivalent reference), since the reported discriminant taxa and LDA threshold behavior are genus-level.
- You need to compare taxonomic profiles against full-length third-generation 16S rRNA sequencing — the study explicitly did not perform this comparison.
- The sample categories are not discrete groups (e.g., region combinations, phenotype labels); LEfSe's LDA-based class discrimination requires group labels.

## Inputs

- Greengenes-collapsed genus-level ASV abundance table (from Deblur-denoised, non-chimeric 16S rRNA reads)
- Grouping/metadata vector assigning each sample to a combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- Region-by-genera assignment table of discriminant genera per hypervariable region
- Ranked list of discriminant genera per region with log10-transformed LDA scores (threshold > 2)

## How to apply

Load the Greengenes-collapsed genus-level ASV abundance table from the authors' GitHub repository, grouped by the four combined hypervariable regions (V1–V2, V3–V4, V5–V7, V7–V9). Scale relative abundances to the range (1, 10exp9) and log10-transform before analysis. Run LEfSe to identify genera differentially abundant among the region groups, using an LDA score threshold > 2, and support the LDA estimates with default 30-fold bootstrapping and averaging to improve biomarker ranking robustness. Rank discriminant genera per region by LDA score to build the region-by-genera assignment table, and cross-check the recovered taxa (e.g., Pseudomonas recovered only in V1–V2; Prevotella, Corynebacterium, Filifactor in V3–V4) against standard-of-care culture results and the ZymoBIOMICS mock community control.

## Related tools

- **Greengenes** (Taxonomic classifier database used to obtain and collapse assignments at the genus level, producing the abundance table that LEfSe consumes)
- **Deblur** (Upstream denoising of paired-end 16S reads to identify bacterial ASVs at the genus level before LEfSe is run)

## Evaluation signals

- Each discriminant genus passes the LDA threshold > 2 only in the region group(s) where it is reported as significant.
- The recovered genus sets reproduce the reported pattern of distinct discriminant taxa per region, e.g., Pseudomonas recovered only for V1–V2, and Prevotella, Corynebacterium, Filifactor among the V3–V4 genera.
- LDA scores are computed after scaling relative abundances to (1, 10exp9) and log10 transformation, with 30-fold bootstrap averaging supporting the ranking.
- Discriminant genera are cross-checked against standard-of-care culture results and consistent findings in the ZymoBIOMICS Microbial Community control.

## Limitations

- The ZymoBIOMICS mock microbial community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification.
- Regions were trained with the outdated Greengenes database despite more updated SILVA versions being available.
- The Illumina MiSeq 300 bp read length limit constrains 16S rRNA profiling across the longer combined regions.
- Discriminant genus sets are region-dependent (distinct sets per region combination), so assignments are not interchangeable across regions.

## Evidence

- [methods] Greengenes genus collapse input: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [abstract] Region discrimination finding: "The combined hypervariable regions showed significant differences when discriminating between the relative abundances of bacterial genera."
- [abstract] Upstream ASV identification: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [results] Mock community control: "Consistent without our findings in the ZymoBIOMICS Microbial Community control (Supplementary Material S1)"
