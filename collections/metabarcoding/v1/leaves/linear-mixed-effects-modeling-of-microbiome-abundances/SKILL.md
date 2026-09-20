---
name: linear-mixed-effects-modeling-of-microbiome-abundances
description: Use when a compositional taxon-by-sample abundance table (e.g., SGB-level
  MetaPhlAn 4 profiles) has repeated/host-structured metadata and the goal is per-taxon
  association testing with covariate control and FDR correction.
when_to_use_negative:
- Input is raw sequencing reads rather than a taxon-by-sample abundance profile —
  profile first with MetaPhlAn 4 before modeling
- Samples lack the covariate structure (e.g., repeated measures, vendor/cohort batching)
  that motivates a random-intercept model; a simple fixed-effect regression would
  suffice
- Taxa below the ~1% mean relative abundance threshold dominate the question (e.g.,
  rare-taxon detection), since sparse CLR-transformed bins lack power here
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Produces the SGB-level relative abundance profiles (including 4,992 taxonomically
    unidentified uSGB bins) that are CLR-transformed and modeled as outcomes
  repo: https://github.com/biobakery/MetaPhlAn
- name: statsmodels MixedLM
  role: Fits the per-SGB random-intercept linear mixed model with diet as fixed effect
    and sex, age, genetic background and vendor as covariates
- name: lme4
  role: Alternative R package for fitting the random-intercept linear mixed models
    per SGB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:21d4eaaba0425119e95df4cc7ad7cf9e7fd45720e719b6cc6668e241744aae72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - lme4
  - statsmodels MixedLM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# linear-mixed-effects-modeling-of-microbiome-abundances

## Summary

Identifies taxonomic biomarkers (e.g., MetaPhlAn 4 SGBs) associated with a host condition by fitting a random-intercept linear mixed model per taxon on CLR-transformed abundances, covarying out host factors and controlling FDR. Used to show that uncharacterized species (uSGBs) dominate high-fat-diet biomarkers in mouse gut microbiomes.

## When to use

Use when a compositional taxon-by-sample abundance table (e.g., SGB-level MetaPhlAn 4 profiles) has repeated/host-structured metadata and the goal is per-taxon association testing with covariate control and FDR correction.

## When NOT to use

- Input is raw sequencing reads rather than a taxon-by-sample abundance profile — profile first with MetaPhlAn 4 before modeling
- Samples lack the covariate structure (e.g., repeated measures, vendor/cohort batching) that motivates a random-intercept model; a simple fixed-effect regression would suffice
- Taxa below the ~1% mean relative abundance threshold dominate the question (e.g., rare-taxon detection), since sparse CLR-transformed bins lack power here

## Inputs

- MetaPhlAn 4 SGB-level relative abundance table (e.g., 181 mouse gut metagenomes)
- Host/sample metadata table: diet group, sex, age, genetic background, vendor/cohort
- SGB-to-taxonomy mapping (to label uSGBs vs characterized species)

## Outputs

- diet_biomarkers_sgb.tsv: SGB biomarker table with effect sizes, p-values, and BH-FDR values (18 SGBs at FDR < 0.2, 13 uSGBs)
- Per-SGB mixed model fits (coefficients and p-values before correction)

## How to apply

Load MetaPhlAn 4 SGB-level relative abundances together with host metadata (e.g., diet group, sex, age, genetic background, vendor), add a pseudocount to zeros, and apply a centered log-ratio (CLR) transform to handle compositionality. Filter taxa to those with mean relative abundance > 1% across the cohort to limit sparse tests. Fit one random-intercept linear mixed model per SGB (lme4 or statsmodels MixedLM), using the condition (e.g., diet HF vs normal chow) as the fixed effect, sex/age/genetic background/vendor as covariates, and a random intercept to absorb cohort non-independence. Correct per-taxon p-values with Benjamini-Hochberg FDR and retain associations at FDR < 0.2, then compile effect sizes, p-values, and FDR values into a TSV table. Classify surviving biomarkers as uSGBs (taxonomically unidentified species bins) to quantify how many are uncharacterized — in the mouse high-fat-diet case, 13 of 18 biomarkers (72%) were uSGBs, including Lachnospiraceae bacterium 28_4 SGB7272, Lactobacillus johnsonii SGB7041, and Faecalibaculum rodentium SGB4047.

## Related tools

- **MetaPhlAn 4** (Produces the SGB-level relative abundance profiles (including 4,992 taxonomically unidentified uSGB bins) that are CLR-transformed and modeled as outcomes) — https://github.com/biobakery/MetaPhlAn
- **statsmodels MixedLM** (Fits the per-SGB random-intercept linear mixed model with diet as fixed effect and sex, age, genetic background and vendor as covariates)
- **lme4** (Alternative R package for fitting the random-intercept linear mixed models per SGB)

## Examples

```
import statsmodels.formula.api as smf; m = smf.mixedlm('clr_sgb_abundance ~ diet + sex + age + genetic_background + vendor', df, groups=df['cohort']).fit()
```

## Evaluation signals

- Output table contains exactly the reported 18 SGB biomarkers called at FDR < 0.2 and mean abundance > 1%
- Anchor SGBs SGB7272 (Lachnospiraceae bacterium 28_4), SGB7041 (Lactobacillus johnsonii), and SGB4047 (Faecalibaculum rodentium) are present in the results
- Majority (~72%, i.e., 13 of 18) of retained biomarkers are classified as uSGBs and over-abundant on the hyper-caloric diet
- All modeled taxa had mean relative abundance > 1% after CLR transform with pseudocount-added zeros
- Benjamini-Hochberg correction applied over the per-SGB p-values with a reported FDR threshold of 0.2

## Limitations

- Compositional abundance data require pseudocounts for zeros and CLR transformation, which can distort sparse taxa
- The relaxed FDR < 0.2 threshold trades specificity for sensitivity, so some called biomarkers may be false positives
- Only taxa with mean relative abundance > 1% are tested, excluding low-abundance biomarkers
- MetaPhlAn 4 databases are updated periodically ('we plan to release at least two new MetaPhlAn databases per year'), so SGB identifiers and uSGB labels may shift across database versions

## Evidence

- [abstract] Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [abstract] uSGB count: "we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [readme] tool purpose: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] biomarker resolution: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
