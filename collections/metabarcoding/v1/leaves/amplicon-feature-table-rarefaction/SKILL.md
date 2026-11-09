---
name: amplicon-feature-table-rarefaction
description: 'Use when downstream alpha/beta diversity comparisons would be confounded
  by unequal sequencing depth: an otutab.txt with heterogeneous sample column sums
  precedes diversity analysis in the EasyAmplicon pipeline.'
when_to_use_negative:
- Input is the raw feature table result/raw/otutab.txt that still contains Eukarya/Chloroplast/Mitochondria/Archaea
  lineages — run otutab_filter_nonBac.R first.
- The analysis plan uses compositional-aware normalization (e.g., DESeq2 size factors
  or cumulative-sum scaling) instead of rarefaction, or the pipeline supports newer
  non-rarefying workflows.
- Samples fall below the chosen rarefaction depth (e.g., <10,000 reads) and would
  be dropped or severely resampled.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: EasyAmplicon
  role: Supplies otutab_rare.R, the vegan-based R script that rarefies the feature
    table to uniform depth
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Provides the companion scripts and databases (e.g., otutab_filter_nonBac.R)
    feeding the cleaned table into rarefaction
  repo: https://github.com/YongxinLiu/EasyMicrobiome
- name: USEARCH
  role: Computes alpha indices (-alpha_div), rarefaction curves (-alpha_div_rare -method
    without_replacement), and validation stats (-otutab_stats) on the rarefied table
- name: vegan (R package)
  role: Implements the sample resampling (rarefaction) routine underlying otutab_rare.R
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_007
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:c6c60e8930f3dbd8f2db1f4e6ee94ec131214dc5063b1f7d9a6a7b643658d1f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - vegan (R package)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# amplicon-feature-table-rarefaction

## Summary

Equalizes sequencing depth across samples by rarefying the EasyAmplicon feature table (result/otutab.txt) with a vegan-based R script plus USEARCH resampling, producing the normalized table (result/otutab_rare.txt) required before alpha diversity, rarefaction curves, and beta diversity analyses.

## When to use

Use when downstream alpha/beta diversity comparisons would be confounded by unequal sequencing depth: an otutab.txt with heterogeneous sample column sums precedes diversity analysis in the EasyAmplicon pipeline.

## When NOT to use

- Input is the raw feature table result/raw/otutab.txt that still contains Eukarya/Chloroplast/Mitochondria/Archaea lineages — run otutab_filter_nonBac.R first.
- The analysis plan uses compositional-aware normalization (e.g., DESeq2 size factors or cumulative-sum scaling) instead of rarefaction, or the pipeline supports newer non-rarefying workflows.
- Samples fall below the chosen rarefaction depth (e.g., <10,000 reads) and would be dropped or severely resampled.

## Inputs

- Cleaned per-sample feature table result/otutab.txt (OTU/ASV rows x sample columns, tab-separated; output of otutab_filter_nonBac.R)
- Sample metadata result/metadata.txt (for confirming sample count matches table columns)
- Rarefaction depth and seed parameters (e.g., --depth 10000 --seed 1)

## Outputs

- Rarefied feature table result/otutab_rare.txt with uniform per-sample depth
- vegan-based diversity metrics result/alpha/vegan.txt
- Alpha diversity index table result/alpha/alpha.txt (14 indices incl. Shannon, Simpson, ACE)
- Rarefaction richness curves result/alpha/alpha_rare.txt (1%–100% sampling depth, without_replacement)

## How to apply

Run the EasyAmplicon R script otutab_rare.R (built on the vegan package's resampling) on result/otutab.txt with --depth 10000 --seed 1, rarefying every sample to a uniform 10,000-read depth with reproducible subsampling; this writes result/otutab_rare.txt and the vegan diversity metrics result/alpha/vegan.txt. Rationale: normalizing for differences in sequencing depth between samples prevents depth-driven bias in subsequent diversity indices. From the rarefied table, compute 14 alpha indices with `usearch -alpha_div` (avoid the tool's miscalculated Chao1) and rarefaction richness curves from 1% to 100% sampling depth with `usearch -alpha_div_rare -method without_replacement`. Beta diversity (Bray-Curtis, UniFrac via the otus.tree) is likewise computed on otutab_rare.txt, so this normalization step is a prerequisite for both diversity axes. Validate with `usearch -otutab_stats` to confirm uniform depth and that the rarefied table retains one column per metadata sample.

## Related tools

- **EasyAmplicon** (Supplies otutab_rare.R, the vegan-based R script that rarefies the feature table to uniform depth) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Provides the companion scripts and databases (e.g., otutab_filter_nonBac.R) feeding the cleaned table into rarefaction) — https://github.com/YongxinLiu/EasyMicrobiome
- **USEARCH** (Computes alpha indices (-alpha_div), rarefaction curves (-alpha_div_rare -method without_replacement), and validation stats (-otutab_stats) on the rarefied table)
- **vegan (R package)** (Implements the sample resampling (rarefaction) routine underlying otutab_rare.R)

## Examples

```
Rscript otutab_rare.R --input result/otutab.txt --depth 10000 --seed 1 --output result/otutab_rare.txt
```

## Evaluation signals

- All four output files exist: result/otutab_rare.txt, result/alpha/vegan.txt, result/alpha/alpha.txt, result/alpha/alpha_rare.txt
- Column count of otutab_rare.txt equals the number of samples in result/metadata.txt
- usearch -otutab_stats on the rarefied table reports uniform sequencing depth across all samples (e.g., 10,000 reads/sample)
- alpha_rare.txt spans sampling fractions from 1% to 100% as produced by -alpha_div_rare
- Re-running with the same --seed 1 reproduces identical rarefied counts (deterministic subsampling)

## Limitations

- Chao1 is miscalculated by the USEARCH -alpha_div tool in this pipeline and should be avoided
- Rarefaction discards reads from deeper-sequenced samples (resampling to 10,000 reads), which is an accepted but data-reducing normalization choice
- Pipeline depends on specific tool versions (USEARCH v10.0.240+, VSEARCH v2.7.1+52); the protocol paper itself provides no changelog or limitations section

## Evidence

- [methods] The pipeline performs sample equalization with an R script that uses the vegan package, producing otutab_rare.txt and vegan.txt: [evidence span withheld — anchor status: not_found]
- [methods] Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample: "Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample"
- [methods] Chao1 is miscalculated by this tool and should be avoided: [evidence span withheld — anchor status: not_found]
- [readme] Equal sampling standardization (rarefaction) [section=methods]: [evidence span withheld — anchor status: not_found]
