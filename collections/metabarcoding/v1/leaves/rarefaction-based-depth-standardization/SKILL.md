---
name: rarefaction-based-depth-standardization
description: Use when comparing per-sample alpha diversity or ASV richness across
  feature tables produced by different denoising or filtering pipelines and uneven
  library sizes need to be equalized before computing Hill Numbers or richness.
when_to_use_negative:
- Input is a raw, unfiltered feature table — the paper's workflow requires applying
  basic/standard/extra filtering (and removing host/mock COI sequences) before rarefaction
- The analysis goal is beta diversity or differential abundance rather than within-sample
  alpha diversity comparison across pipeline choices
- Sample libraries cannot reach the 5,000-read depth, since standard filtering already
  retains only samples with ≥5,000 total filtered reads and rarefaction would drop
  them
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Phyloseq
  role: R package used to rarefy filtered feature tables to 5,000 reads per sample
    and compute Hill Number alpha diversity on the guano data
- name: Vegan
  role: R package used alongside Phyloseq to compute Hill Numbers 0, 1, and 2 for
    every guano sample
- name: Tidyverse
  role: R framework used for the Kruskal–Wallis and Dunn's post hoc tests and for
    plotting the per-sample diversity figures
- name: QIIME 2
  role: Environment (v2018.11) in which the raw paired-end guano reads were imported,
    trimmed, denoised, and filtered prior to rarefaction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:b5716abbcea956a4bc23319be13fb678e33ae9c2904072f170068858c479c99c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Deblur
  - Phyloseq
  - QIIME 2
  - Tidyverse
  - Vegan
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# rarefaction-based-depth-standardization

## Summary

Standardizes filtered metabarcoding feature tables to a fixed read depth (5,000 reads per sample for the bat guano COI data in O'Rourke et al. 2020) before computing per-sample alpha diversity (Hill Numbers 0, 1, 2), so that diversity comparisons among denoising pipelines (DADA2, Deblur, VSEARCH) and filtering regimes are not confounded by sequencing depth.

## When to use

Use when comparing per-sample alpha diversity or ASV richness across feature tables produced by different denoising or filtering pipelines and uneven library sizes need to be equalized before computing Hill Numbers or richness.

## When NOT to use

- Input is a raw, unfiltered feature table — the paper's workflow requires applying basic/standard/extra filtering (and removing host/mock COI sequences) before rarefaction
- The analysis goal is beta diversity or differential abundance rather than within-sample alpha diversity comparison across pipeline choices
- Sample libraries cannot reach the 5,000-read depth, since standard filtering already retains only samples with ≥5,000 total filtered reads and rarefaction would drop them

## Inputs

- Filtered QIIME 2 feature tables (basic, standard, and extra filtering regimes) from bat guano COI amplicon data (SRA PRJNA518082), imported into R
- Denoised/clustering representative-sequence tables from DADA2, Deblur (--p-min-reads 2, --p-min-size 1), and VSEARCH (98% then 97% identity clustering) pipelines
- Rarefaction depth threshold (5,000 reads per sample)

## Outputs

- Rarefied feature tables at 5,000 reads per sample for each denoising x filtering combination
- Per-sample Hill Number 0, 1, and 2 values (observed richness, Shannon's Entropy, Simpson's Index)
- Kruskal–Wallis and Dunn's post hoc test results for diversity differences among denoising programs
- Figure 3 / Figure S2-style plots of richness and diversity by denoising program and filtering regime

## How to apply

After denoising/clustering and applying a filtering regime (basic, standard — dropping single-sample sequence variants and retaining samples with ≥5,000 filtered reads — or extra, which additionally subtracts a per-library fixed integer defined as the maximum count of a 'miss' mock sequence variant), rarefy each filtered feature table to 5,000 reads per sample. Compute Hill Numbers 0, 1, and 2 (observed richness, Shannon's Entropy, Simpson's Index) for every sample using Phyloseq/Vegan in R. Compare diversity values across denoising programs with Kruskal–Wallis tests per filtering regime and Hill Number, followed by Dunn's post hoc pairwise tests. Plot per-sample estimates by program and regime to visualize whether diversity remains sensitive to the bioinformatic pipeline after standardization. This matters because the paper found VSEARCH-processed samples repeatedly contained 2–3x more ASVs per sample than DADA2 or Deblur, a gap rarefaction helps place on equal footing rather than eliminate.

## Related tools

- **Phyloseq** (R package used to rarefy filtered feature tables to 5,000 reads per sample and compute Hill Number alpha diversity on the guano data)
- **Vegan** (R package used alongside Phyloseq to compute Hill Numbers 0, 1, and 2 for every guano sample)
- **Tidyverse** (R framework used for the Kruskal–Wallis and Dunn's post hoc tests and for plotting the per-sample diversity figures)
- **QIIME 2** (Environment (v2018.11) in which the raw paired-end guano reads were imported, trimmed, denoised, and filtered prior to rarefaction)

## Examples

```
rarefied <- phyloseq::rarefy_even_depth(guano_physeq, sample.size = 5000, rngseed = 711)
```

## Evaluation signals

- Rarefied tables must have exactly 5,000 reads per retained sample; samples below depth are excluded, matching the ≥5,000-read standard filter criterion
- Hill Number 0 for basic-filtered data should be the one case that does NOT differ significantly among denoising programs; all other Hill Number x filtering combinations should show significant Kruskal–Wallis differences
- VSEARCH-processed samples should show roughly 2–3x more ASVs per sample than DADA2 or Deblur in the richness estimates
- Dunn's post hoc tests should identify which pairwise program contrasts drive the Kruskal–Wallis significance for each filtering regime
- Plotted per-sample diversity patterns must reproduce the Figure 3 / Figure S2 layout (diversity by program across basic/standard/extra regimes)

## Limitations

- Rarefaction discards reads and samples, so absolute richness is underestimated relative to the non-rarefied tables; the paper shows denoising program effects persist even after this standardization
- Hill Number 0 on basic-filtered data is not sensitive to denoising method, so the standardization does not guarantee detection of pipeline effects for every metric/regime combination
- Results are specific to the 181 bp COI fragment and bat guano dataset; the authors state their aim is not to present a single best pathway for all animal metabarcoding projects
- Software versions for the denoising programs were not reported in the results section, complicating exact reproduction

## Evidence

- [other] Rarefy each filtered dataset to 5,000 reads per sample and compute Hill Numbers 0, 1, and 2 (observed richness, Shannon's Entropy, Simpson's Index) for every guano sample using Phyloseq/Vegan in R.: [evidence span withheld — anchor status: not_found]
- [methods] Standard filter definition with 5,000-read threshold: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [abstract] Abstract finding on denoising vs clustering error rates: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [readme] README pointing to diversity analysis documentation: "sequence_and_diversity_analyses.md"
