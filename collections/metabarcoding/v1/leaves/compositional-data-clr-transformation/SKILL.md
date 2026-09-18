---
name: compositional-data-clr-transformation
description: Use when a compositional abundance table (SGB/species relative abundances)
  must be log-transformed before per-taxon statistical testing, e.g. fitting diet-association
  mixed models on mouse gut MetaPhlAn 4 profiles.
when_to_use_negative:
- Input is already a log-transformed or absolute-count feature table rather than compositional
  relative abundances
- Profiling has not yet been run — raw shotgun reads must first be converted to a
  SGB abundance profile (e.g. with MetaPhlAn 4) before CLR can be applied
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: generates the compositional SGB-level relative abundance profiles that are
    CLR-transformed
  repo: https://github.com/biobakery/MetaPhlAn
- name: statsmodels
  role: MixedLM used to fit per-SGB random-intercept models on CLR-transformed abundances
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:9674e7a907f77e3d1c593ae5f7c8a7263918d664b4bac409fbfee95b6b01f227
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - statsmodels
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Compositional data CLR transformation

## Summary

Applies a centered log-ratio (CLR) transform to compositional taxonomic abundance tables (e.g. MetaPhlAn 4 SGB relative abundances), adding a pseudocount to zeros before the log transform so that abundance data become suitable for downstream differential-abundance modeling such as linear mixed models.

## When to use

Use when a compositional abundance table (SGB/species relative abundances) must be log-transformed before per-taxon statistical testing, e.g. fitting diet-association mixed models on mouse gut MetaPhlAn 4 profiles.

## When NOT to use

- Input is already a log-transformed or absolute-count feature table rather than compositional relative abundances
- Profiling has not yet been run — raw shotgun reads must first be converted to a SGB abundance profile (e.g. with MetaPhlAn 4) before CLR can be applied

## Inputs

- MetaPhlAn 4 SGB-level relative abundance table for the cohort (e.g. 181 mouse gut metagenomes)
- Host/metadata table (diet group, sex, age, genetic background, vendor)

## Outputs

- CLR-transformed SGB abundance matrix
- Per-SGB association results table (effect sizes, p-values, FDR) e.g. diet_biomarkers_sgb.tsv

## How to apply

Load the MetaPhlAn 4 SGB-level relative abundance profiles (e.g. the 181 mouse gut metagenomes) with host metadata such as diet, sex, age, genetic background and vendor. Because the table is compositional, apply a centered log-ratio (CLR) transform, adding a pseudocount to zero abundances prior to the log transform to avoid log(0). Optionally filter SGBs to those with mean relative abundance greater than 1% across the cohort before or after transformation to keep the tested feature set tractable. Fit per-SGB models (e.g. random-intercept linear mixed models with diet HF vs chow as fixed effect via lme4 or statsmodels MixedLM), then Benjamini-Hochberg correct p-values (here retaining FDR < 0.2) to obtain biomarkers; 18 SGBs including 13 uSGBs such as SGB7272, SGB7041 and SGB4047 were expected in the source analysis.

## Related tools

- **MetaPhlAn 4** (generates the compositional SGB-level relative abundance profiles that are CLR-transformed) — https://github.com/biobakery/MetaPhlAn
- **statsmodels** (MixedLM used to fit per-SGB random-intercept models on CLR-transformed abundances)

## Examples

```
import numpy as np; clr = np.log((X + 1e-6) / (X + 1e-6).sum(axis=1, keepdims=True))
```

## Evaluation signals

- Output biomarker table contains the reported 18 SGB biomarkers called at FDR < 0.2 and mean abundance > 1%
- The three anchor SGBs (SGB7272, SGB7041, SGB4047) are present in the output table
- Majority (~72%, 13 of 18) of retained biomarkers are classified as uncharacterized species (uSGBs)
- Zero abundances were replaced with a pseudocount before the log transform (no NaN/inf values in the CLR matrix)
- Columns of the CLR matrix sum (approximately) to zero per sample, confirming the CLR was applied correctly

## Limitations

- Under-characterized habitats remain enriched for sequences not captured by current uSGBs, so CLR results on such communities may miss real taxa
- Tradeoff between sensitivity and specificity when applying quality control to identify real but rare taxa remains an unaddressed downstream statistical challenge

## Evidence

- [other] CLR transform on SGB abundances with pseudocount: [evidence span withheld — anchor status: not_found]
- [abstract] uSGB biomarker finding: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [readme] MetaPhlAn profiling scope: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
