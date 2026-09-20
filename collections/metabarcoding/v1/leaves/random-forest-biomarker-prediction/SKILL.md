---
name: random-forest-biomarker-prediction
description: Use when shotgun metagenomes have been profiled with MetaPhlAn 4 SGBs
  and the goal is to predict host health/dietary markers or quantify how uncharacterized
  taxa improve prediction over a prior profile version.
when_to_use_negative:
- Input is functional (UniRef/pathway) profile data rather than MetaPhlAn taxonomic
  SGB profiles
- Samples come from under-characterized, non-host-associated, or environmental habitats,
  where sequences are still highly enriched for content not captured by current uSGBs
- The question is strain-level population genomics rather than species-level biomarker
  prediction
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0625
tools:
- name: MetaPhlAn 4
  role: Generates the SGB/uSGB relative abundance profiles used as predictive features;
    its expanded marker set (26,970 SGBs) is what drives prediction improvement over
    MetaPhlAn 3
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Provides the baseline (MetaPhlAn 3) species-level taxonomic profiles against
    which random forest prediction gains are benchmarked
  repo: https://github.com/biobakery/MetaPhlAn
- name: bioBakery
  role: Suite/platform containing MetaPhlAn and supporting tutorials for the microbial
    community profiling step feeding the models
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:fe7ce392d839c464d035b459f14f1ddd368ee93e5b00e142f311735b681fb9d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - bioBakery
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# random-forest-biomarker-prediction

## Summary

Train random forest classifiers/regressors on MetaPhlAn 4 species-level genome bin (SGB) relative abundance profiles to predict nutritional and cardiometabolic health markers, and benchmark the gains from uncharacterized taxa (uSGBs) against an older MetaPhlAn 3-based profiling of the same samples.

## When to use

Use when shotgun metagenomes have been profiled with MetaPhlAn 4 SGBs and the goal is to predict host health/dietary markers or quantify how uncharacterized taxa improve prediction over a prior profile version.

## When NOT to use

- Input is functional (UniRef/pathway) profile data rather than MetaPhlAn taxonomic SGB profiles
- Samples come from under-characterized, non-host-associated, or environmental habitats, where sequences are still highly enriched for content not captured by current uSGBs
- The question is strain-level population genomics rather than species-level biomarker prediction

## Inputs

- MetaPhlAn 4 SGB relative abundance matrix (samples x species-level genome bins, including uSGBs)
- MetaPhlAn 3-based SGB relative abundance matrix for the same samples (baseline)
- Panel of host nutritional/cardiometabolic marker measurements (e.g. 19 markers: aMED, HEI, uPDI, 10-year ASCVD risk)
- Covariates for partial Spearman correlation adjustment

## Outputs

- Per-marker cross-validated AUC/R2 distributions over 100 bootstrap iterations
- Delta AUC and percent improvement per marker (MetaPhlAn 4 vs MetaPhlAn 3)
- Partial Spearman correlations between uSGB abundances and dietary/health markers
- Aggregated results table (e.g. marker_prediction_results.csv)

## How to apply

Load the SGB relative abundance matrix (e.g. 1,001 ZOE PREDICT 1 individuals profiled with MetaPhlAn 4, which defines markers for 26,970 SGBs, 4,992 of them taxonomically unidentified) together with the panel of host markers (19 nutritional/cardiometabolic measurements). For each marker, train a random forest classifier/regressor using 80/20 train-test splits repeated over 100 bootstrap iterations, recording cross-validated AUC (or R2) per split. Repeat the identical modeling procedure on the MetaPhlAn 3-based SGB profiles and compute per-marker delta AUC and percent improvement. Then compute partial Spearman correlations between individual uSGB abundances (e.g. Alphaproteobacteria SGB4777) and markers to attribute predictive gains to specific uncharacterized species. Rationale: MetaPhlAn 4's expanded SGB set explains ~20% more reads in human gut microbiomes, so previously undetected species can act as strong host-condition biomarkers.

## Related tools

- **MetaPhlAn 4** (Generates the SGB/uSGB relative abundance profiles used as predictive features; its expanded marker set (26,970 SGBs) is what drives prediction improvement over MetaPhlAn 3) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Provides the baseline (MetaPhlAn 3) species-level taxonomic profiles against which random forest prediction gains are benchmarked) — https://github.com/biobakery/MetaPhlAn
- **bioBakery** (Suite/platform containing MetaPhlAn and supporting tutorials for the microbial community profiling step feeding the models) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan && metaphlan metagenome.sam --input_type sam --profile_vscg -o sgb_profile.tsv
```

## Evaluation signals

- Median cross-validated AUC across the 19 markers should reach the anchor value 0.74, a 4.84% improvement over the MetaPhlAn 3 baseline
- Largest per-marker gain should be 10-year ASCVD risk (+0.106 AUC, 16.24% improvement)
- Strongest dietary association should be the Healthy Eating Index score (+0.072 AUC, 10.05% AUC improvement and 31% regression improvement)
- SGB4777 (Alphaproteobacteria) should appear among the strongest correlated uSGBs with partial Spearman correlations aMED rho=0.21, HEI rho=0.19, uPDI rho=-0.25
- Per-marker delta AUC and percent improvement columns must be computable for all 19 markers from paired bootstrap distributions

## Limitations

- Further work remains to better profile under-characterized habitats even with the extended MetaPhlAn 4 SGB and marker set
- Environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured by current uSGBs
- Tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge
- MetaPhlAn databases expand over time (at least two new releases per year planned), so baseline comparisons are version-dependent

## Evidence

- [abstract] From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [readme] To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [discussion] Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats.: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
- [discussion] Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
