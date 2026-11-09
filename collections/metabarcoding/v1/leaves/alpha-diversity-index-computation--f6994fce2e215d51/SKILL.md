---
name: alpha-diversity-index-computation
description: Use when a normalized (rarefied) OTU/feature table with uniform per-sample
  depth (e.g. 10,000 reads) is available and alpha diversity indices plus rarefaction
  curves are needed for downstream diversity analysis and visualization.
when_to_use_negative:
- The OTU table is already rarefied to uniform depth and you only need beta diversity
  or ordination
- You need reliable per-sample Chao1 estimates — USEARCH's Chao1 is known to be miscalculated
  in this pipeline and should be avoided
- Working with unnormalized raw counts of highly variable sequencing depth where rarefaction
  to 10,000 reads would discard excessive data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: USEARCH
  role: Computes the 14 alpha diversity indices (-alpha_div), rarefaction richness
    curves (-alpha_div_rare), and table statistics (-otutab_stats) on the rarefied
    table
- name: vegan
  role: R package underlying otutab_rare.R, which resamples all samples to a uniform
    read depth (--depth 10000 --seed 1)
- name: EasyAmplicon
  role: Provides otutab_rare.R and the surrounding pipeline scripts and output-directory
    conventions (result/alpha/)
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:639b399bddddb91131c564dd75c08ae54af51b7c73b58d5aa285d85b637443da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - USEARCH
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# alpha-diversity-index-computation

## Summary

Compute within-sample alpha diversity indices and rarefaction richness curves in the EasyAmplicon pipeline, operating on a rarefied OTU table (otutab_rare.txt) produced by the vegan-based equalization script so that sequencing-depth differences do not bias diversity estimates.

## When to use

Use when a normalized (rarefied) OTU/feature table with uniform per-sample depth (e.g. 10,000 reads) is available and alpha diversity indices plus rarefaction curves are needed for downstream diversity analysis and visualization.

## When NOT to use

- The OTU table is already rarefied to uniform depth and you only need beta diversity or ordination
- You need reliable per-sample Chao1 estimates — USEARCH's Chao1 is known to be miscalculated in this pipeline and should be avoided
- Working with unnormalized raw counts of highly variable sequencing depth where rarefaction to 10,000 reads would discard excessive data

## Inputs

- result/otutab.txt — raw OTU abundance table (OTUs × samples)
- result/otutab_rare.txt — vegan-rarefied table resampled to a uniform depth (e.g. 10,000 reads, seed 1)

## Outputs

- result/otutab_rare.txt — normalized equal-depth OTU table
- result/alpha/vegan.txt — diversity metrics from the vegan rarefaction script
- result/alpha/alpha.txt — table of 14 alpha diversity indices per sample (Shannon, Simpson, ACE, ...)
- result/alpha/alpha_rare.txt — rarefaction richness values across 1%–100% sampling depths

## How to apply

First ensure the input table is depth-standardized: run the EasyAmplicon R script otutab_rare.R (built on the vegan package) on result/otutab.txt with --depth 10000 --seed 1, writing result/otutab_rare.txt and vegan.txt. Confirm uniform depth with `usearch -otutab_stats` before proceeding. Then compute 14 alpha diversity indices (Shannon, Simpson, ACE, etc.; avoid Chao1 because USEARCH miscalculates it in this pipeline) with `usearch -alpha_div` on otutab_rare.txt, writing result/alpha/alpha.txt. Generate rarefaction richness data across 1%–100% sampling depth with `usearch -alpha_div_rare` using -method without_replacement, writing result/alpha/alpha_rare.txt. These outputs feed the downstream alpha-diversity plotting step.

## Related tools

- **USEARCH** (Computes the 14 alpha diversity indices (-alpha_div), rarefaction richness curves (-alpha_div_rare), and table statistics (-otutab_stats) on the rarefied table)
- **vegan** (R package underlying otutab_rare.R, which resamples all samples to a uniform read depth (--depth 10000 --seed 1))
- **EasyAmplicon** (Provides otutab_rare.R and the surrounding pipeline scripts and output-directory conventions (result/alpha/)) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
usearch -alpha_div result/otutab_rare.txt -output result/alpha/alpha.txt
```

## Evaluation signals

- All four output files exist: otutab_rare.txt, vegan.txt, alpha.txt, and alpha_rare.txt
- The rarefied table's sample columns match the input sample count (no samples lost during rarefaction)
- `usearch -otutab_stats` on otutab_rare.txt confirms uniform sequencing depth across all samples
- alpha.txt contains the expected index set (Shannon, Simpson, ACE, etc.); Chao1 values are treated as unreliable and excluded
- alpha_rare.txt spans richness values from 1% to 100% sampling depth, generated with method without_replacement

## Limitations

- Chao1 is miscalculated by USEARCH's -alpha_div in this pipeline and should be avoided as a metric
- Rarefaction to a fixed depth (e.g. 10,000 reads with seed 1) discards reads from deeper samples and depends on the chosen depth and seed
- VSEARCH (used elsewhere in the pipeline for clustering/dechimerization) supports subsampling but USEARCH-specific -alpha_div options must run via USEARCH itself

## Evidence

- [methods] Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample: "Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample"
- [full_text] Compute 14 alpha diversity indices (e.g., Shannon, Simpson, ACE; note Chao1 is miscalculated by this tool and should be avoided) with `usearch -alpha_div result/otutab_rare.txt -output result/alpha/alpha.txt`.: [evidence span withheld — anchor status: not_found]
- [full_text] Generate the rarefaction richness data from 1% to 100% sampling depth with `usearch -alpha_div_rare result/otutab_rare.txt -output result/alpha/alpha_rare.txt -method without_replacement`.: [evidence span withheld — anchor status: not_found]
- [readme] EasyAmplicon: An easy-to-use, open-source, reproducible, and community-based pipeline for amplicon data analysis in microbiome research.: "EasyAmplicon: An easy-to-use, open-source, reproducible, and community-based pipeline for amplicon data analysis in microbiome research."
