---
name: usgb-marker-partial-spearman-correlation
description: Use when MetaPhlAn 4 SGB/uSGB abundance profiles need to be linked to
  dietary (aMED, HEI, uPDI) or cardiometabolic markers via covariate-adjusted (partial)
  Spearman rank correlations to identify association biomarkers.
when_to_use_negative:
- Input communities are environmental, nonhost-associated, or under-studied habitats,
  which the paper notes remain highly enriched for sequences not captured by current
  uSGBs
- The goal is predictive performance benchmarking (AUC/R² of random forests) rather
  than ranking individual microbe–marker associations
- Abundances come from a profiler that does not report uncharacterized SGBs (e.g.
  MetaPhlAn 3), since the expected signals (e.g. SGB4777) are uSGB-specific
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Generates the SGB/uSGB relative abundance profiles that are correlated against
    the dietary and cardiometabolic markers
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Baseline profiler whose earlier SGB profiles provide the comparison baseline
    (MetaPhlAn 3) in the marker-association re-analysis
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:e606bac2dc5d90717ab6a695016eee022b310c6488585240070c0013ba44eedc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# usgb-marker-partial-spearman-correlation

## Summary

Computes partial Spearman correlations between MetaPhlAn 4 uncharacterized species-level genome bin (uSGB) relative abundances and host dietary/cardiometabolic markers to rank microbe–health associations, as re-analyzed in the ZOE PREDICT 1 cohort (n=1,001) where SGB4777 (Alphaproteobacteria) emerged as the top signal.

## When to use

Use when MetaPhlAn 4 SGB/uSGB abundance profiles need to be linked to dietary (aMED, HEI, uPDI) or cardiometabolic markers via covariate-adjusted (partial) Spearman rank correlations to identify association biomarkers.

## When NOT to use

- Input communities are environmental, nonhost-associated, or under-studied habitats, which the paper notes remain highly enriched for sequences not captured by current uSGBs
- The goal is predictive performance benchmarking (AUC/R² of random forests) rather than ranking individual microbe–marker associations
- Abundances come from a profiler that does not report uncharacterized SGBs (e.g. MetaPhlAn 3), since the expected signals (e.g. SGB4777) are uSGB-specific

## Inputs

- MetaPhlAn 4 SGB/uSGB relative abundance matrix (samples × SGBs, e.g. 1,001 ZOE PREDICT 1 individuals)
- Panel of nutritional and cardiometabolic marker measurements per sample (aMED, HEI, uPDI, ASCVD risk, etc.)
- Covariate/confounder table for computing partial (adjusted) correlations

## Outputs

- Per-uSGB, per-marker partial Spearman correlation coefficients and ranks
- Aggregated results table (marker_prediction_results.csv) combining uSGB–marker correlations with per-marker model performance deltas

## How to apply

First generate species-level relative abundance profiles with MetaPhlAn 4, whose marker set covers 26,970 species-level genome bins including 4,992 taxonomically unidentified uSGBs, yielding an SGB-by-sample abundance matrix (e.g. 1,001 ZOE PREDICT 1 individuals). Extract the uSGB rows and, together with the panel of marker measurements, compute partial Spearman correlations so that rank association strengths are adjusted for confounding covariates. Correlate each uSGB against each dietary/health marker (aMED, HEI, uPDI, etc.), ranking uSGBs by correlation magnitude per marker; in the ZOE PREDICT 1 re-analysis the expected top uSGB was Alphaproteobacteria SGB4777 with aMED ρ=0.21, HEI ρ=0.19, and uPDI ρ=-0.25. Assemble the per-uSGB, per-marker correlation coefficients alongside any random-forest AUC comparison deltas (e.g. vs MetaPhlAn 3 profiles) into a single results table (marker_prediction_results.csv). The rationale is that uSGBs capture previously undetected taxa that the paper showed to be strong biomarkers for host conditions and lifestyles.

## Related tools

- **MetaPhlAn 4** (Generates the SGB/uSGB relative abundance profiles that are correlated against the dietary and cardiometabolic markers) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Baseline profiler whose earlier SGB profiles provide the comparison baseline (MetaPhlAn 3) in the marker-association re-analysis) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- Alphaproteobacteria SGB4777 should appear among the strongest correlated uSGBs across markers
- Anchor correlation values reproduce: aMED ρ=0.21, HEI ρ=0.19, uPDI ρ=-0.25 for SGB4777
- All reported coefficients are valid Spearman statistics within [-1, 1], computed after covariate adjustment
- Results table (marker_prediction_results.csv) contains both uSGB–marker correlations and per-marker AUC deltas (median AUC 0.74, +4.84% vs MetaPhlAn 3; ASCVD +0.106; HEI +0.072)
- Number of uSGBs tested matches the MetaPhlAn 4 abundance matrix rows (4,992 taxonomically unidentified SGBs available in the marker set)

## Limitations

- Many related downstream statistical challenges remain, such as the tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa
- Uncharacterized uSGBs often have no cultured isolates, so correlation signals cannot be tied to reference-annotated biology without additional phylogenetic characterization (e.g. PhyloPhlAn)
- Environmental and under-studied microbial communities are still highly enriched for sequences not captured by current uSGBs, limiting generalization beyond host-associated cohorts

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins including 4,992 taxonomically unidentified uSGBs used as correlation features: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [abstract] uSGBs are strong biomarkers for host conditions and lifestyles, motivating correlation against host dietary/health markers: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [readme] MetaPhlAn provides the species-level SGB abundance profiles that serve as the correlation input: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [abstract] MetaPhlAn 4 integrates assemblies and isolate genomes to profile organisms with no cultured isolates, which underlies the uSGB feature space: "MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes for more comprehensive metagenomic taxonomic profiling."
