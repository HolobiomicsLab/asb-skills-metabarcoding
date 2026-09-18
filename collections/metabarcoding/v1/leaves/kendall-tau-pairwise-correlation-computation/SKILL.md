---
name: kendall-tau-pairwise-correlation-computation
description: Use when you have a proportional-abundance taxon-by-sample time-series
  (e.g. 18 monthly eDNA samples across 663 taxonomic groups) plus aligned environmental
  measurements, and need non-parametric rank correlations with FDR-controlled significance
  across many pairwise tests.
when_to_use_negative:
- 'When the goal is to assert direct biological interactions: the article warns that
  co-occurrence may be ''a common response to an external driver'', and network connectivity
  does not necessarily indicate interaction'
- When the abundance table is not on a proportional-abundance scale or the time-series
  is much shorter/longer than the 18-month monthly design, since the tau = 0.70 threshold
  was calibrated on this specific dataset and its 100-permutation null
- When a parametric correlation method on normally distributed variables would suffice
  — Kendall tau is chosen here because abundances are rank-based and non-normal
tools:
- name: pandas
  role: Load the 18-month abundance table and aligned environmental-property measurements
    from the repository
- name: scipy.stats.kendalltau
  role: Compute each pairwise Kendall tau rank correlation on the proportional-abundance
    time-series
- name: numpy
  role: Column-wise permutation shuffling of abundance vectors and recomputation of
    pairwise taus to build the empirical null density
- name: statsmodels.stats.multitest.multipletests
  role: Benjamini–Hochberg false-discovery-rate adjustment of the empirical p-values
- name: networkx
  role: Downstream community-network analysis of the significant tau pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:e3746600b9f1f2b550d10fbd52cc2b2b606f993039d81a26a313b500a5bb5490
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - networkx
  - numpy
  - pandas
  - scipy
  - scipy.stats.kendalltau
  - statsmodels
  - statsmodels.stats.multitest.multipletests
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.38461538461538464
n_claims: 13
---

# kendall-tau-pairwise-correlation-computation

## Summary

Computes all pairwise Kendall tau rank correlations between taxon eDNA abundance indices and environmental properties across an 18-month Monterey Bay time-series, and controls the false-discovery rate over ~6.5×10^4 comparisons via a 100-permutation null model plus Benjamini–Hochberg adjustment, yielding a corrected significance threshold of tau = 0.70.

## When to use

Use when you have a proportional-abundance taxon-by-sample time-series (e.g. 18 monthly eDNA samples across 663 taxonomic groups) plus aligned environmental measurements, and need non-parametric rank correlations with FDR-controlled significance across many pairwise tests.

## When NOT to use

- When the goal is to assert direct biological interactions: the article warns that co-occurrence may be 'a common response to an external driver', and network connectivity does not necessarily indicate interaction
- When the abundance table is not on a proportional-abundance scale or the time-series is much shorter/longer than the 18-month monthly design, since the tau = 0.70 threshold was calibrated on this specific dataset and its 100-permutation null
- When a parametric correlation method on normally distributed variables would suffice — Kendall tau is chosen here because abundances are rank-based and non-normal

## Inputs

- Proportional-abundance eDNA taxon-by-sample time-series table (18 monthly seawater samples, 663 taxonomic groups at Family or higher)
- Aligned environmental-property measurements (e.g. sea surface temperature, chlorophyll a)
- 100-permutation null distribution of pairwise Kendall tau values (column-wise shuffles of abundance vectors)

## Outputs

- fdr_tau_results.csv with columns: taxon pair, tau, empirical p-value, BH-adjusted p-value, significance flag (significant where corrected tau >= 0.70)
- Corrected significance threshold (tau = 0.70) feeding the community-network analysis

## How to apply

Load the 18-month Monterey Bay eDNA multilocus amplicon abundance table and aligned environmental-property measurements (sea surface temperature, chlorophyll a, etc.) from github.com/marinebon/eDNA_microbes_whales with pandas. Compute every pairwise Kendall tau between the ~6.5×10^4 taxon/environmental variable pairs on the proportional-abundance time-series using scipy.stats.kendalltau. Build an empirical null by column-wise shuffling of the abundance vectors 100 times and recomputing the full pairwise tau set per permutation with numpy, then derive empirical p-values by locating each observed tau within the 100-permutation null density. Apply statsmodels.stats.multitest.multipletests (Benjamini–Hochberg FDR adjustment, equivalent to a Bonferroni-style multiple-testing correction at this scale) and flag pairs as significant only where the corrected statistic clears tau = 0.70. Rationale: with thousands of simultaneous rank correlations on short monthly time-series, nominal tau p-values would be dominated by false positives, so a permutation null plus FDR correction is used before the community-network analysis.

## Related tools

- **pandas** (Load the 18-month abundance table and aligned environmental-property measurements from the repository)
- **scipy.stats.kendalltau** (Compute each pairwise Kendall tau rank correlation on the proportional-abundance time-series)
- **numpy** (Column-wise permutation shuffling of abundance vectors and recomputation of pairwise taus to build the empirical null density)
- **statsmodels.stats.multitest.multipletests** (Benjamini–Hochberg false-discovery-rate adjustment of the empirical p-values)
- **networkx** (Downstream community-network analysis of the significant tau pairs)

## Examples

```
from scipy.stats import kendalltau; tau, p = kendalltau(df[taxon_a], df[taxon_b])  # per pair on proportional abundances; FDR via statsmodels.multipletests
```

## Evaluation signals

- Reproduced corrected significance threshold equals tau = 0.70 exactly
- Number of significant pairs retained after FDR adjustment matches the community-network analysis reported in the paper
- Empirical null is built from 100 column-wise permutations and empirical p-values fall in [0, 1] consistent with that density
- Benjamini–Hochberg adjusted p-values are monotone with empirical p-values and no pair is flagged significant with tau < 0.70
- Output CSV schema contains taxon pair, tau, empirical p-value, adjusted p-value, and significance flag for every pairwise combination

## Limitations

- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Network connectivity does not necessarily indicate interaction; hypotheses from network analyses require further hypothesis-based research and experiments
- Amplification is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, affecting which taxa enter the correlation set
- eDNA does not allow identification of life history stages and could include sequences from dead or decaying biological matter
- Longer time-series are needed to increase the power and validity of the technique

## Evidence

- [abstract] We inferred changes in the composition of communities, revealing putative interactions among taxa and identifying correlations between these communities and environmental properties over time.: "revealing putative interactions among taxa and identifying correlations between these communities and environmental properties over time"
- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [discussion] whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
