---
name: time-series-co-occurrence-significance-thresholding
description: Use when a rank-based co-occurrence network built from proportional-abundance
  time-series (taxa × environmental properties) needs multiple-testing-corrected significance
  thresholds, e.g. thousands of pairwise Kendall tau comparisons.
when_to_use_negative:
- When the goal is taxonomic detection or abundance estimation itself rather than
  significance-thresholding of co-occurrence statistics — the article notes amplification
  bias, inhibitors, and variable template abundances affect taxa detection, and 'no
  method—including this one—identifies all taxa in an ecosystem'
- When the time-series is too short to support 100-permutation null estimation with
  enough power — the article states 'longer time-series needed to increase power and
  validity'
- 'When interpreting network edges as confirmed biological interactions: ''Network
  connectivity does not necessarily indicate interaction'''
tools:
- name: pandas
  role: Load the 18-month eDNA multilocus amplicon abundance table and aligned environmental-property
    measurements from the github.com/marinebon/eDNA_microbes_whales repository
  repo: github.com/marinebon/eDNA_microbes_whales
- name: scipy
  role: Compute all pairwise Kendall tau rank correlations on the proportional-abundance
    time-series via scipy.stats.kendalltau
  repo: github.com/marinebon/eDNA_microbes_whales
- name: numpy
  role: Column-wise shuffling of abundance vectors and recomputation of pairwise Kendall
    taus per permutation to build the empirical null density
  repo: github.com/marinebon/eDNA_microbes_whales
- name: statsmodels
  role: Apply Benjamani-Hochberg false-discovery-rate adjustment to empirical p-values
    via statsmodels.stats.multitest.multipletests
  repo: github.com/marinebon/eDNA_microbes_whales
- name: networkx
  role: Build the community-network analysis (subnetworks) from the significant tau
    pairs that evidence predator-prey relationships and trophic linkages
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:d907668369dd5ca48a4e70af1fff83fcccb7914dea10c96758e3a717d99a9aae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - Python
  - networkx
  - numpy
  - pandas
  - scipy
  - statsmodels
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# time-series-co-occurrence-significance-thresholding

## Summary

A permutation null-model procedure that assigns empirical p-values to pairwise Kendall's tau rank correlations computed over an 18-month Monterey Bay eDNA proportional-abundance time-series and applies Benjamani-Hochberg false-discovery-rate correction, yielding a corrected significance threshold of tau = 0.70.

## When to use

Use when a rank-based co-occurrence network built from proportional-abundance time-series (taxa × environmental properties) needs multiple-testing-corrected significance thresholds, e.g. thousands of pairwise Kendall tau comparisons.

## When NOT to use

- When the goal is taxonomic detection or abundance estimation itself rather than significance-thresholding of co-occurrence statistics — the article notes amplification bias, inhibitors, and variable template abundances affect taxa detection, and 'no method—including this one—identifies all taxa in an ecosystem'
- When the time-series is too short to support 100-permutation null estimation with enough power — the article states 'longer time-series needed to increase power and validity'
- When interpreting network edges as confirmed biological interactions: 'Network connectivity does not necessarily indicate interaction'

## Inputs

- 18-month (2015–2016) Monterey Bay seawater eDNA multilocus amplicon abundance table covering 663 taxonomic groups (Family or higher)
- Aligned environmental-property measurements (sea surface temperature, chlorophyll a, and other key environmental and biotic variables)
- Proportional-abundance time-series vectors for each taxon and environmental variable

## Outputs

- Corrected pairwise results table (taxon pair, tau, empirical p-value, adjusted p-value, significance flag) written to fdr_tau_results.csv
- Set of significant taxon/environmental pairs (corrected tau ≥ 0.70) used as edges in the community-network analysis
- Empirical null distribution of pairwise Kendall taus from 100 column-wise permutations

## How to apply

Compute all pairwise Kendall tau rank correlations across the taxon/environmental variable pairs of the proportional-abundance time-series with scipy.stats.kendalltau. Build an empirical null distribution by permuting the abundance vectors column-wise 100 times and recomputing the full set of pairwise taus per permutation with numpy. Locate each observed tau within the null density to derive empirical p-values, then adjust them with statsmodels.stats.multitest.multipletests (Benjamani-Hochberg false-discovery-rate adjustment, equivalent to a Bonferroni-style correction at this scale). Flag pairs as significant only where the corrected statistic clears the reported threshold of tau = 0.70, and feed the retained significant pairs into the community-network analysis (networkx) that identifies predator-prey relationships, trophic linkages, and seasonal shifts.

## Related tools

- **pandas** (Load the 18-month eDNA multilocus amplicon abundance table and aligned environmental-property measurements from the github.com/marinebon/eDNA_microbes_whales repository) — github.com/marinebon/eDNA_microbes_whales
- **scipy** (Compute all pairwise Kendall tau rank correlations on the proportional-abundance time-series via scipy.stats.kendalltau) — github.com/marinebon/eDNA_microbes_whales
- **numpy** (Column-wise shuffling of abundance vectors and recomputation of pairwise Kendall taus per permutation to build the empirical null density) — github.com/marinebon/eDNA_microbes_whales
- **statsmodels** (Apply Benjamani-Hochberg false-discovery-rate adjustment to empirical p-values via statsmodels.stats.multitest.multipletests) — github.com/marinebon/eDNA_microbes_whales
- **networkx** (Build the community-network analysis (subnetworks) from the significant tau pairs that evidence predator-prey relationships and trophic linkages) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
from scipy.stats import kendalltau; import statsmodels.stats.multitest as mt; tau, p = kendalltau(taxon_abundance, environmental_variable); adj = mt.multipletests(p, method='fdr_bh')
```

## Evaluation signals

- The reproduced corrected significance threshold equals tau = 0.70 after FDR adjustment of empirical p-values
- The number of significant pairs retained after Benjamani-Hochberg adjustment matches the community-network analysis reported in the paper
- The empirical null density contains 100 permutations of the full pairwise tau set, with column-wise (not row-wise) shuffling of abundance vectors
- fdr_tau_results.csv contains the expected columns: taxon pair, tau, empirical p-value, adjusted p-value, significance flag
- Observed taus falling within the bulk of the 100-permutation null density are correctly flagged non-significant (no significant pairs with small tau)

## Limitations

- Co-occurrence significance does not confirm mechanism: 'whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear'
- Significant network edges require follow-up: 'These analyses can predict taxon interdependencies that should be further investigated through hypothesis-based research and experiments.'
- Taxon detection limits propagate into the correlation set: 'Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA'
- Lower taxonomic assignment reduces confidence in the resulting pairwise correlations, since 'the lower the taxonomy assignment, the lower the confidence in that assignment becomes'

## Evidence

- [abstract] permutation and network workflow: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [abstract] correlation target data: "We inferred changes in the composition of communities, revealing putative interactions among taxa and identifying correlations between these communities and environmental properties over time."
- [abstract] time-series provenance: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [discussion] interpretation caveat for significant co-occurrence edges: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [readme] repository role for downstream analysis: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
