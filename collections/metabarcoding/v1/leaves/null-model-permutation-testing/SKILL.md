---
name: null-model-permutation-testing
description: 'Use when an observed metric (e.g., NODF nestedness of a presence-absence
  matrix) must be tested against chance: build null matrices via constrained randomization,
  then derive z-scores and permutation p-values from many permutations.'
when_to_use_negative:
- The input is already a randomized/null matrix or you only need to recompute a p-value
  from an existing null distribution
- The analysis goal is differential abundance or diversity estimation (e.g., alpha/beta
  diversity, SourceTracker2 source-sinking) rather than testing whether an observed
  matrix pattern exceeds a null model
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0610
tools:
- name: Python
  role: Implements the NODF computation, matrix randomization under the equiprobable-rows/fixed-columns
    null model, z-score and permutation p-value calculation, and CSV export
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Pandas
  role: Loads the EMP BIOM table and mapping file and constructs the binary presence-absence
    matrices by taxonomic level and environment
  repo: https://github.com/biocore/emp/tree/master/methods
- name: redbiom
  role: Optionally retrieves the sample data and metadata used to assemble the matrices
    (e.g., 'redbiom fetch samples --context' into BIOM)
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:c6bce7c572719aebd0f5167132c3f6b99f1b1c8f7a688ef05bd745015f996d86
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - redbiom
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# null-model-permutation-testing

## Summary

Computes permutation-based null distributions of an ecological statistic (here, NODF nestedness on binary presence-absence matrices of EMP 16S samples) to produce z-scores and p-values that test whether observed community structure exceeds chance.

## When to use

Use when an observed metric (e.g., NODF nestedness of a presence-absence matrix) must be tested against chance: build null matrices via constrained randomization, then derive z-scores and permutation p-values from many permutations.

## When NOT to use

- The input is already a randomized/null matrix or you only need to recompute a p-value from an existing null distribution
- The analysis goal is differential abundance or diversity estimation (e.g., alpha/beta diversity, SourceTracker2 source-sinking) rather than testing whether an observed matrix pattern exceeds a null model

## Inputs

- EMP 16S Release 1 BIOM table for the 2,000-sample subset
- EMP mapping file (sample metadata with environment types: non-saline, saline, animal, plant)
- Binary presence-absence matrices aggregated at phylum, class, or order level

## Outputs

- Empirical NODF statistic per matrix and per environment subset
- Z-scores relative to the null distribution
- Permutation p-values
- nodf_results.csv containing all NODF statistics and p-values

## How to apply

Convert the filtered BIOM abundance table (EMP 16S Release 1, chloroplast sequences excluded, mc2 minimum-sequence-count filter applied) into binary presence-absence matrices aggregated at the phylum, class, or order level with pandas. Compute the empirical NODF (Nestedness based on Overlap and Decreasing Fill) on each matrix and on per-environment subsets (non-saline, saline, animal, plant). Generate null distributions by randomizing each matrix under an equiprobable-rows/fixed-columns null model with many permutations; the choice of null model is the key decision point because it fixes which marginal constraints are preserved. Compute empirical NODF, z-scores, and permutation p-values, then declare significance where P < 0.05 relative to the null distribution.

## Related tools

- **Python** (Implements the NODF computation, matrix randomization under the equiprobable-rows/fixed-columns null model, z-score and permutation p-value calculation, and CSV export) — https://github.com/biocore/emp/tree/master/methods
- **Pandas** (Loads the EMP BIOM table and mapping file and constructs the binary presence-absence matrices by taxonomic level and environment) — https://github.com/biocore/emp/tree/master/methods
- **redbiom** (Optionally retrieves the sample data and metadata used to assemble the matrices (e.g., 'redbiom fetch samples --context' into BIOM)) — https://github.com/biocore/redbiom

## Examples

```
import numpy as np; z=(nodf_obs-np.mean(nodf_null))/np.std(nodf_null); p=(np.sum(nodf_null>=nodf_obs)+1)/(len(nodf_null)+1)
```

## Evaluation signals

- Overall NODF falls within 0.841 ± 0.018 (95% CI) for the 2,000-sample subset
- Per-environment NODF reproduces ~0.811 (non-saline), ~0.787 (saline), ~0.788 (animal), ~0.860 (plant), each significant at P < 0.05 versus the null
- Overall significance reaches P < 0.0001 against the equiprobable-rows/fixed-columns null model
- Every analyzed matrix has a reported z-score and a permutation p-value derived from many permutations
- All NODF statistics and p-values are written to nodf_results.csv

## Limitations

- Results depend on the null model choice: the equiprobable-rows/fixed-columns randomization preserves column (taxon) frequencies but randomizes row fill, so significance can shift under alternative nulls
- Preprocessing filters materially affect the matrices: chloroplast sequences must be excluded and the mc2 minimum-sequence-count filter (minimum sequence count in an OTU to be included equals 2) applied before binarization
- Analysis uses binary presence-absence after aggregation to phylum/class/order, discarding abundance information and being sensitive to the chosen taxonomic level

## Evidence

- [methods] Generate null distributions by randomizing each matrix under an equiprobable-rows/fixed-columns null model with many permutations.: [evidence span withheld — anchor status: not_found]
- [results] NODF result and significance: [evidence span withheld — anchor status: not_found]
- [supplementary] Pre-analysis filtering: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [readme] redbiom data retrieval: [evidence span withheld — anchor status: external_doc]
