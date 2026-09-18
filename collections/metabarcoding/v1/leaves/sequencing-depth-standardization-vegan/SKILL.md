---
name: sequencing-depth-standardization-vegan
description: Use when an amplicon feature/OTU table (e.g., result/otutab.txt) has
  uneven sequencing depths across samples and downstream alpha diversity or rarefaction
  analyses require equal sampling before comparison.
when_to_use_negative:
- The input table is already rarefied or samples are at uniform depth — rarefying
  again would only discard data
- The goal is compositional normalization for beta diversity or differential abundance,
  where methods like DESeq2 size factors or relative abundance transformation are
  preferred over rarefaction
- Chao1 is needed downstream — the article notes Chao1 is miscalculated by this tooling
  and should be avoided
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: vegan
  role: R package whose resampling routine performs the equal-depth rarefaction of
    the OTU table
- name: EasyAmplicon
  role: Provides otutab_rare.R, the R script that wraps vegan rarefaction in the pipeline
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: USEARCH
  role: Consumes the rarefied table for alpha_div, alpha_div_rare, and otutab_stats
    validation
- name: EasyMicrobiome
  role: Supplies the bundled scripts and software environment used by EasyAmplicon
    for this normalization step
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:4c6d33dd1baa05ef115b328d5291603df8041a727a85a3e2ce82795c17c2779f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - USEARCH
  - vegan
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# sequencing-depth-standardization-vegan

## Summary

Equalize sequencing depth across samples by rarefying the OTU/ASV count table with an R script built on the vegan package (otutab_rare.R), producing a uniformly sampled table that feeds alpha diversity and rarefaction analyses in the EasyAmplicon pipeline.

## When to use

Use when an amplicon feature/OTU table (e.g., result/otutab.txt) has uneven sequencing depths across samples and downstream alpha diversity or rarefaction analyses require equal sampling before comparison.

## When NOT to use

- The input table is already rarefied or samples are at uniform depth — rarefying again would only discard data
- The goal is compositional normalization for beta diversity or differential abundance, where methods like DESeq2 size factors or relative abundance transformation are preferred over rarefaction
- Chao1 is needed downstream — the article notes Chao1 is miscalculated by this tooling and should be avoided

## Inputs

- result/otutab.txt (OTU/ASV abundance table, features x samples, with unequal per-sample read counts)

## Outputs

- result/otutab_rare.txt (rarefied table at uniform sequencing depth, e.g., 10,000 reads per sample)
- result/alpha/vegan.txt (vegan-based diversity metrics per sample)

## How to apply

Run the EasyAmplicon R script otutab_rare.R, which uses the vegan package to resample all samples to a uniform depth (e.g., --depth 10000 with --seed 1 for reproducibility), writing the rarefied table to result/otutab_rare.txt and vegan-based diversity metrics to result/alpha/vegan.txt. The rarefied table then becomes the input for computing 14 alpha diversity indices with `usearch -alpha_div` and rarefaction richness curves from 1% to 100% depth with `usearch -alpha_div_rare -method without_replacement`. Validate by confirming all output files exist, that the rarefied table retains the same sample count as the input, and by running `usearch -otutab_stats` on the resampled table to confirm uniform sequencing depth. Rationale: normalizing for differences in sequencing depth between samples removes read-count bias before diversity metrics are compared.

## Related tools

- **vegan** (R package whose resampling routine performs the equal-depth rarefaction of the OTU table)
- **EasyAmplicon** (Provides otutab_rare.R, the R script that wraps vegan rarefaction in the pipeline) — https://github.com/YongxinLiu/EasyAmplicon
- **USEARCH** (Consumes the rarefied table for alpha_div, alpha_div_rare, and otutab_stats validation)
- **EasyMicrobiome** (Supplies the bundled scripts and software environment used by EasyAmplicon for this normalization step) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
Rscript otutab_rare.R --input result/otutab.txt --depth 10000 --seed 1 --output result/otutab_rare.txt; usearch -otutab_stats result/otutab_rare.txt
```

## Evaluation signals

- All four outputs exist: result/otutab_rare.txt, result/alpha/vegan.txt, result/alpha/alpha.txt, and result/alpha/alpha_rare.txt
- Sample columns in otutab_rare.txt match the sample count of the input otutab.txt (no samples lost)
- `usearch -otutab_stats` on the rarefied table reports uniform sequencing depth across all samples
- Re-running with the same --seed (e.g., --seed 1) reproduces the identical rarefied table
- Downstream alpha_div output contains the expected 14 alpha diversity indices (Shannon, Simpson, ACE, etc.)

## Limitations

- The USEARCH alpha_div Chao1 calculation is known to be miscalculated and should be avoided when computing alpha indices from the rarefied table
- Rarefaction discards reads from deeper-sequenced samples, so choosing too low a depth (e.g., 10,000) throws away usable data for high-depth samples

## Evidence

- [methods] vegan rarefaction workflow: "Normalize for differences in sequencing depth between samples. Use the R package vegan script to resample"
- [full_text] workflow card finding: [evidence span withheld — anchor status: not_found]
- [full_text] usearch alpha_div command: [evidence span withheld — anchor status: not_found]
- [methods] otutab_trim filter step: "usearch -otutab_trim result/otutab_rare.txt -min_otu_freq 0.005 -output tree/otutab.txt"
