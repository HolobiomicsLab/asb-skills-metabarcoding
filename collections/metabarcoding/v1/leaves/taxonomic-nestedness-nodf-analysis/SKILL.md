---
name: taxonomic-nestedness-nodf-analysis
description: Use when you have a BIOM abundance table plus sample metadata (e.g.,
  EMP environment types non-saline/saline/animal/plant) and need to test whether taxa
  at phylum, class, or order level are gained but not lost with increasing sample
  richness.
when_to_use_negative:
- 'Raw count matrices: the workflow binarizes to presence-absence before computing
  NODF, so applying it directly to raw abundances or rarefied counts misrepresents
  the metric'
- Questions about beta diversity, ordination, or source tracking (e.g., SourceTracker2
  use cases) rather than nestedness of community composition
- Datasets lacking environment-type metadata or with too few taxa per sample to support
  meaningful null-model permutations
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Implements the NODF computation, equiprobable-rows/fixed-columns null-model
    permutations, z-scores, and permutation p-values on the EMP matrices
  repo: https://github.com/biocore/emp
- name: Pandas
  role: Loads the BIOM table and mapping file for the 2,000-sample subset and handles
    the presence-absence conversion and per-environment subsetting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:78eab739dc954a971007f2da0a46069892c2422675e8a1bcafd5ff303c2175b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/emp
  tools:
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# taxonomic-nestedness-nodf-analysis

## Summary

Quantifies whether microbial community taxonomic composition is nested across environments by computing the NODF (Nestedness based on Overlap and Decreasing Fill) statistic on binary presence-absence matrices from EMP 16S Release 1, tested against an equiprobable-rows/fixed-columns null model. It reproduces the finding that EMP communities are significantly nested (overall NODF = 0.841 ± 0.018, P < 0.0001).

## When to use

Use when you have a BIOM abundance table plus sample metadata (e.g., EMP environment types non-saline/saline/animal/plant) and need to test whether taxa at phylum, class, or order level are gained but not lost with increasing sample richness.

## When NOT to use

- Raw count matrices: the workflow binarizes to presence-absence before computing NODF, so applying it directly to raw abundances or rarefied counts misrepresents the metric
- Questions about beta diversity, ordination, or source tracking (e.g., SourceTracker2 use cases) rather than nestedness of community composition
- Datasets lacking environment-type metadata or with too few taxa per sample to support meaningful null-model permutations

## Inputs

- EMP 16S Release 1 BIOM abundance table (2,000-sample subset)
- EMP mapping file with environment-type metadata (non-saline, saline, animal, plant)
- Taxonomic aggregation level choice: phylum, class, or order

## Outputs

- nodf_results.csv with empirical NODF values, z-scores, and permutation p-values per taxonomic level and per environment subset
- Null distributions of NODF from equiprobable-rows/fixed-columns randomizations

## How to apply

Load the EMP 16S Release 1 BIOM table and mapping file for the 2,000-sample subset into Python with pandas, restricting to non-chloroplast taxa and applying the mc2 minimum-sequence-count filter (minimum sequence count in an OTU to be included equals 2). Convert abundances to binary presence-absence matrices aggregated at the phylum, class, and order levels. Compute NODF on each full matrix and on per-environment subsets (non-saline, saline, animal, plant). Generate null distributions by randomizing each matrix under an equiprobable-rows/fixed-columns null model with many permutations, then compute empirical NODF, z-scores, and permutation p-values. Validation requires overall NODF within 0.841 ± 0.018 and significant per-environment values at P < 0.05; results are written to nodf_results.csv.

## Related tools

- **Python** (Implements the NODF computation, equiprobable-rows/fixed-columns null-model permutations, z-scores, and permutation p-values on the EMP matrices) — https://github.com/biocore/emp
- **Pandas** (Loads the BIOM table and mapping file for the 2,000-sample subset and handles the presence-absence conversion and per-environment subsetting)

## Evaluation signals

- Overall NODF across the 2,000-sample subset falls within 0.841 ± 0.018 (95% CI)
- Overall significance holds versus the null model at P < 0.0001
- Per-environment NODF values reproduce the published results: 0.811 non-saline, 0.787 saline, 0.788 animal, 0.860 plant, each significant at P < 0.05
- Chloroplast-annotated sequences are excluded before statistics are computed, and the mc2 minimum-sequence-count filter (count >= 2) is applied
- nodf_results.csv is written with NODF statistics and p-values for every taxonomic level and environment subset analyzed

## Limitations

- Results are specific to the EMP 16S Release 1 meta-analysis of the first 97 16S rRNA amplicon studies and the 2,000-sample subset; other releases (e.g., Release 2) are not covered
- NODF is computed on binary presence-absence matrices, so abundance information is discarded and conclusions concern presence/loss patterns only
- Permutation-based p-values depend on the chosen null model (equiprobable-rows/fixed-columns) and the number of permutations; a different null model can change significance

## Evidence

- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed.: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [readme] EMP 16S Release 1 scope: "EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published"
- [supplementary] Rarefaction of samples: "Samples were rarefied to 5,000 observations per sample."
