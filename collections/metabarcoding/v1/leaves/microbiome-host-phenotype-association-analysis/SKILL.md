---
name: microbiome-host-phenotype-association-analysis
description: Use when metagenomic shotgun samples have been profiled with MetaPhlAn
  4 and you must link taxonomic abundances (e.g. SGB4777) to host markers like HEI,
  aMED, uPDI or ASCVD risk via prediction metrics.
when_to_use_negative:
- Input is raw shotgun reads not yet profiled at species level — first generate relative
  abundances with MetaPhlAn 4 before linking to host markers
- Markers are of a single continuous type with no classification target and no baseline
  profile — the AUC-based benchmarking against MetaPhlAn 3 has no anchor
- Study is an environmental or nonhost-associated community — the article notes such
  communities remain highly enriched for sequences not captured by current uSGBs,
  undermining host-phenotype association power
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: MetaPhlAn 4
  role: Generates the species-level/SGB relative abundance profiles used as predictors
    of host dietary and cardiometabolic markers
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Provides the prior MetaPhlAn 3-based SGB baseline profiles against which MetaPhlAn
    4 prediction improvement is computed
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:f4e9211df2d6a2a095bc59fc9aa07fe25839fa1e2278694745cacda65969915d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# microbiome-host-phenotype-association-analysis

## Summary

Relates MetaPhlAn 4 taxonomic profiles (SGB/uSGB relative abundances) to host nutritional and cardiometabolic markers by cross-validated random forest prediction and partial Spearman correlation, benchmarking MetaPhlAn 4 against MetaPhlAn 3 baselines.

## When to use

Use when metagenomic shotgun samples have been profiled with MetaPhlAn 4 and you must link taxonomic abundances (e.g. SGB4777) to host markers like HEI, aMED, uPDI or ASCVD risk via prediction metrics.

## When NOT to use

- Input is raw shotgun reads not yet profiled at species level — first generate relative abundances with MetaPhlAn 4 before linking to host markers
- Markers are of a single continuous type with no classification target and no baseline profile — the AUC-based benchmarking against MetaPhlAn 3 has no anchor
- Study is an environmental or nonhost-associated community — the article notes such communities remain highly enriched for sequences not captured by current uSGBs, undermining host-phenotype association power

## Inputs

- MetaPhlAn 4 SGB/uSGB relative abundance matrix (e.g. for 1,001 ZOE PREDICT 1 individuals)
- MetaPhlAn 3-based SGB relative abundance profiles (comparison baseline)
- Host nutritional and cardiometabolic marker measurements (19 markers incl. HEI, aMED, uPDI, 10-year ASCVD risk)

## Outputs

- Per-marker median cross-validated AUC/R2 values
- Delta AUC and percent improvement table vs MetaPhlAn 3 baseline
- Partial Spearman correlations between uSGB abundances and dietary/health markers
- marker_prediction_results.csv

## How to apply

Load the MetaPhlAn 4 SGB relative abundance matrix (here 1,001 ZOE PREDICT 1 individuals) alongside the panel of 19 nutritional/cardiometabolic markers. Train random forest classifiers/regressors per marker on 80/20 train-test splits repeated over 100 bootstrap iterations, recording cross-validated AUC/R2 per split; repeat identically on MetaPhlAn 3-based SGB profiles to compute per-marker delta AUC and percent improvement (expected medians: 0.74 AUC overall, +4.84%; ASCVD +0.106/16.24%; HEI +0.072/10.05%). As a complementary association view, compute partial Spearman correlations between uSGB abundances and dietary/health markers (Alphaproteobacteria SGB4777 expected as top signal: aMED ρ=0.21, HEI ρ=0.19, uPDI ρ=-0.25). Assemble per-marker median AUCs, MetaPhlAn 3 deltas, and uSGB-marker correlations into a single results table (marker_prediction_results.csv).

## Related tools

- **MetaPhlAn 4** (Generates the species-level/SGB relative abundance profiles used as predictors of host dietary and cardiometabolic markers) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Provides the prior MetaPhlAn 3-based SGB baseline profiles against which MetaPhlAn 4 prediction improvement is computed) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- Median AUC across the 19 markers matches the anchor value of 0.74 (a 4.84% improvement over the MetaPhlAn 3 baseline)
- Largest per-marker gain corresponds to 10-year ASCVD risk (+0.106 AUC, 16.24%)
- Strongest dietary association is the Healthy Eating Index score (+0.072 AUC, 10.05% improvement; 31% regression improvement)
- SGB4777 (Alphaproteobacteria) appears among the strongest correlated uSGBs with aMED ρ=0.21, HEI ρ=0.19, uPDI ρ=-0.25
- marker_prediction_results.csv contains per-marker median AUCs, MetaPhlAn 3 deltas, and uSGB-marker partial Spearman correlations

## Limitations

- MetaPhlAn 4's extended SGB set still leaves under-characterized habitats poorly profiled; environmental and nonhost-associated communities remain enriched for sequences not captured by current uSGBs
- A tradeoff between sensitivity and specificity under quality-control measures for real but rare taxa remains an unaddressed downstream statistical challenge
- Prediction gains are specific to the ZOE PREDICT 1 cohort (n=1,001) and the 19-marker panel; deltas may not transfer to other cohorts

## Evidence

- [abstract] MetaPhlAn 4 quantifies uncharacterized taxa for biomarker discovery: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [readme] MetaPhlAn scope: species-level profiling from shotgun data: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] MetaPhlAn 4 enables improved biomarker resolution: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
- [discussion] Environmental communities remain hard to profile: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
