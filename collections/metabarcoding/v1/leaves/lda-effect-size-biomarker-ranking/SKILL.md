---
name: lda-effect-size-biomarker-ranking
description: Use when a genus-level ASV abundance table split into predefined classes
  (e.g., hypervariable region combinations) must be screened for differentially abundant
  taxa and ranked as biomarkers by LDA effect size.
when_to_use_negative:
- Input is raw reads or full-length 16S rRNA sequence data rather than a collapsed
  genus-level abundance table with predefined sample classes
- The analysis goal is overall diversity or ordination (e.g., alpha diversity by Kruskal–Wallis,
  NMDS of Bray–Curtis dissimilarities) rather than group-discriminating taxa
- No meaningful class variable exists — LEfSe requires predefined groups (e.g., region
  combinations) to estimate between-class effect sizes
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: Greengenes
  role: Reference database used to obtain genus-level taxonomic assignments of the
    ASV table that LEfSe analyzes; the region classifiers were trained with Greengenes
- name: LEfSe
  role: Effect-size method that assigns discriminant genera per hypervariable-region
    group using LDA scores (threshold > 2, log10-transformed, 30-fold bootstrap averaging)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:337048dd3226655280b69f9c4b3ef7046712c6c2a0801bf7078d8a9ac2866bb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Greengenes
  - LEfSe
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lda-effect-size-biomarker-ranking

## Summary

Runs LEfSe on Greengenes-collapsed, genus-level 16S rRNA ASV abundance tables to rank discriminant bacterial genera per sample group (here, combined hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9) using log10-transformed LDA scores with a threshold > 2.

## When to use

Use when a genus-level ASV abundance table split into predefined classes (e.g., hypervariable region combinations) must be screened for differentially abundant taxa and ranked as biomarkers by LDA effect size.

## When NOT to use

- Input is raw reads or full-length 16S rRNA sequence data rather than a collapsed genus-level abundance table with predefined sample classes
- The analysis goal is overall diversity or ordination (e.g., alpha diversity by Kruskal–Wallis, NMDS of Bray–Curtis dissimilarities) rather than group-discriminating taxa
- No meaningful class variable exists — LEfSe requires predefined groups (e.g., region combinations) to estimate between-class effect sizes

## Inputs

- Genus-level ASV abundance table collapsed with the Greengenes database (e.g., Greengenes-collapsed QIIME 2 table)
- Class/metadata column defining sample groups (combined hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9)
- Optional external validation sets: ZymoBIOMICS mock community control and standard-of-care culture results

## Outputs

- Ranked list of discriminant genera per region group with log10-transformed LDA scores (only genera with LDA > 2 retained)
- Region-by-genera assignment table mapping each biomarker to the region group(s) where it passes the threshold
- LEfSe cladogram showing compositional relationships between the tested region combinations

## How to apply

Load the Greengenes-collapsed genus-level ASV abundance table and group samples by the class of interest (here the four combined hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9). Scale relative abundances to the range (1, 10exp9) before LEfSe so the effect-size computation is numerically stable. Run LEfSe with an LDA significance threshold > 2 applied to log10-transformed LDA scores, and enable the default 30-fold bootstrapping with averaging so biomarker ranking is robust. Assign each genus to the region group(s) where it passes the LDA > 2 threshold, producing a region-by-genera table; in this study each combination yielded a distinct genus set (V1–V2 uniquely recovered Pseudomonas, plus Glesbergeria, Sinobaca, Ochromonas; V3–V4 recovered Prevotella, Corynebacterium, Filifactor, among others). Finally, cross-check recovered biomarkers against external evidence such as standard-of-care culture results and the ZymoBIOMICS mock community control, and inspect the cladogram for compositional similarity between groups.

## Related tools

- **Greengenes** (Reference database used to obtain genus-level taxonomic assignments of the ASV table that LEfSe analyzes; the region classifiers were trained with Greengenes)
- **LEfSe** (Effect-size method that assigns discriminant genera per hypervariable-region group using LDA scores (threshold > 2, log10-transformed, 30-fold bootstrap averaging))

## Examples

```
format_input.py genus_by_region.txt -c 1 -o 1000000000; lefse.py genus_by_region.in genus_by_region.out -l 2
```

## Evaluation signals

- Every reported biomarker genus has a log10-transformed LDA score above the 2.0 threshold; genera below threshold are excluded from the region-by-genera table
- The genus sets are distinct per region combination — V1–V2 is the only group recovering Pseudomonas, while V3–V4 yields Prevotella, Corynebacterium, Filifactor, among others
- The 30-fold bootstrap default was applied so that LDA score rankings are averages, not single-run estimates
- Recovered discriminant genera are consistent with external checks: standard-of-care culture results and the ZymoBIOMICS mock community control
- The cladogram for V3–V4 and V5–V7 shows very similar composition, matching the reported compositional dissimilarity pattern (V1–V2 and V7–V9 versus V3–V4 and V5–V7)

## Limitations

- Taxonomic assignments rely on the outdated Greengenes database rather than more updated versions of SILVA, which can affect which genera are called discriminant
- The ZymoBIOMICS mock community contains only 10 species, which affects the sensitivity and specificity of taxonomic classification used for validation
- Discriminant genus assignments are niche- and region-dependent for sputum samples from patients with chronic respiratory disease; richness and entropy vary with actual composition

## Evidence

- [abstract] The combined hypervariable regions showed significant differences when discriminating between the relative abundances of bacterial genera.: "The combined hypervariable regions showed significant differences when discriminating between the relative abundances of bacterial genera."
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level.: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [full_text] using an LDA threshold > 2 after scaling relative abundances to (1, 10exp9) and computing log10-transformed LDA scores: [evidence span withheld — anchor status: not_found]
- [discussion] despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes: "despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes"
