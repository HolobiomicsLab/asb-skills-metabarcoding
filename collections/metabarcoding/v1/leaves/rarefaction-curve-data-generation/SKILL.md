---
name: rarefaction-curve-data-generation
description: Use when sequencing depth varies across samples, you have already rarefied
  the OTU table (e.g. otutab_rare.txt at 10,000 reads), and you need richness-vs-sampling-fraction
  data for rarefaction curves.
when_to_use_negative:
- Input is the raw (non-rarefied) OTU table result/otutab.txt — samples at unequal
  sequencing depth would bias richness curves; rarefy first with otutab_rare.R
- You only need per-sample alpha diversity indices (Shannon, Simpson, ACE) at full
  depth — use `usearch -alpha_div` instead of -alpha_div_rare
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: USEARCH
  role: Runs -alpha_div_rare with -method without_replacement on the rarefied table
    to produce the 1–100% rarefaction richness data
- name: EasyAmplicon
  role: Pipeline containing the otutab_rare.R script (vegan-based) that produces the
    rarefied input table otutab_rare.txt
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:a07c9556552b4163d1422a53c9efb523f7aae83094a090f3d0a85795536b2315
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rarefaction-curve-data-generation

## Summary

Generate rarefaction richness data from the rarefied OTU table using USEARCH's alpha_div_rare, sampling from 1% to 100% of reads with without_replacement so downstream richness-vs-depth curves can be plotted.

## When to use

Use when sequencing depth varies across samples, you have already rarefied the OTU table (e.g. otutab_rare.txt at 10,000 reads), and you need richness-vs-sampling-fraction data for rarefaction curves.

## When NOT to use

- Input is the raw (non-rarefied) OTU table result/otutab.txt — samples at unequal sequencing depth would bias richness curves; rarefy first with otutab_rare.R
- You only need per-sample alpha diversity indices (Shannon, Simpson, ACE) at full depth — use `usearch -alpha_div` instead of -alpha_div_rare

## Inputs

- Rarefied OTU/ASV table at uniform sequencing depth (result/otutab_rare.txt, e.g. all samples at 10,000 reads)
- Depth-normalized table produced by the vegan-based R script otutab_rare.R

## Outputs

- Rarefaction richness table (result/alpha/alpha_rare.txt) with per-sample richness from 1% to 100% sampling depth
- Data used for rarefaction curve plots in alpha diversity visualization

## How to apply

After normalizing read counts across samples to uniform depth (EasyAmplicon's otutab_rare.R, built on the vegan package, rarefies result/otutab.txt to --depth 10000 with --seed 1, producing result/otutab_rare.txt), run USEARCH's rarefaction command on the rarefied table: `usearch -alpha_div_rare result/otutab_rare.txt -output result/alpha/alpha_rare.txt -method without_replacement`. The without_replacement method resamples reads without putting them back, which is the appropriate subsampling model for richness estimation. Output spans 1% to 100% sampling depth per sample, yielding the per-sample richness values plotted as rarefaction curves in downstream alpha diversity visualizations. Validate first that otutab_rare.txt exists and samples are at uniform depth, e.g. via `usearch -otutab_stats`, before rarefying curves.

## Related tools

- **USEARCH** (Runs -alpha_div_rare with -method without_replacement on the rarefied table to produce the 1–100% rarefaction richness data)
- **EasyAmplicon** (Pipeline containing the otutab_rare.R script (vegan-based) that produces the rarefied input table otutab_rare.txt) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
usearch -alpha_div_rare result/otutab_rare.txt -output result/alpha/alpha_rare.txt -method without_replacement
```

## Evaluation signals

- result/alpha/alpha_rare.txt exists and is non-empty after the run
- Rarefied input table passes `usearch -otutab_stats`, confirming uniform sequencing depth across samples
- Rarefied table's sample columns match the input sample count before rarefaction
- Rarefaction values span the 1% to 100% sampling-depth range for every sample
- At 100% sampling depth, per-sample richness in alpha_rare.txt is consistent with richness from the full-depth alpha index table

## Limitations

- Chao1 is miscalculated by this EasyAmplicon tooling and should be avoided when interpreting richness values
- Rarefaction requires the input table to already be depth-normalized (vegan-based otutab_rare.R at a fixed --depth, e.g. 10000 with --seed 1); running on an uneven-depth table gives biased curves
- without_replacement subsampling discards reads rather than resampling, so low-depth samples contribute fewer curve points

## Evidence

- [methods] Rarefaction richness data: [evidence span withheld — anchor status: not_found]
- [methods] Rarefy to uniform depth: [evidence span withheld — anchor status: not_found]
- [methods] Equal sampling standardization: "Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample"
- [methods] Validation of uniform depth: [evidence span withheld — anchor status: not_found]
