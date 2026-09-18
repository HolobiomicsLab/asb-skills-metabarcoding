---
name: benjamini-hochberg-false-discovery-rate-correction
description: Use when a large set of pairwise rank correlations (e.g., Kendall tau
  between eDNA taxon abundance series and environmental properties) must be significance-tested
  simultaneously and raw p-values would inflate false discoveries.
when_to_use_negative:
- When only a small number of hypotheses are tested and raw p-values with a simple
  alpha cutoff suffice — permutation + FDR machinery is unnecessary overhead.
- When the input p-values are already FDR- or Bonferroni-adjusted; re-applying multipletests
  would double-correct and under-report significant pairs.
- When correlations are not rank-based pairwise statistics (e.g., OTU co-abundance
  networks built from different association measures with their own null models).
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: statsmodels
  role: Apply Benjamini–Hochberg false-discovery-rate adjustment to empirical p-values
    via statsmodels.stats.multitest.multipletests
- name: numpy
  role: Column-wise permutation of abundance time-series and recomputation of the
    pairwise tau matrix to build the empirical null density
- name: scipy
  role: Compute pairwise Kendall tau rank correlations on proportional-abundance time-series
    with scipy.stats.kendalltau
- name: pandas
  role: Load the 18-month abundance table and aligned environmental measurements from
    the eDNA_microbes_whales repository
  repo: github.com/marinebon/eDNA_microbes_whales
- name: networkx
  role: Downstream construction of the community network from FDR-significant tau
    pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:29121a5a286a16072a6da60a3012d4ba513c60a7dc129634901591b2b97283ab
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.125
n_claims: 8
---

# benjamini-hochberg-false-discovery-rate-correction

## Summary

Controls the false-discovery rate across the ~6.5×10^4 pairwise Kendall tau correlations computed between taxon/environmental abundance time-series in the Monterey Bay eDNA dataset, yielding a corrected significance threshold of tau = 0.70. It combines a permutation null (100 time-series permutations) with Benjamini–Hochberg adjustment of empirical p-values.

## When to use

Use when a large set of pairwise rank correlations (e.g., Kendall tau between eDNA taxon abundance series and environmental properties) must be significance-tested simultaneously and raw p-values would inflate false discoveries.

## When NOT to use

- When only a small number of hypotheses are tested and raw p-values with a simple alpha cutoff suffice — permutation + FDR machinery is unnecessary overhead.
- When the input p-values are already FDR- or Bonferroni-adjusted; re-applying multipletests would double-correct and under-report significant pairs.
- When correlations are not rank-based pairwise statistics (e.g., OTU co-abundance networks built from different association measures with their own null models).

## Inputs

- Proportional-abundance time-series table for 663 taxonomic groups (Family or higher) from the 18-month Monterey Bay seawater eDNA multilocus amplicon dataset (2015–2016)
- Aligned environmental-property time-series (sea surface temperature, chlorophyll a, and other key environmental and biotic variables)
- Observed pairwise Kendall tau correlation matrix across taxon/environmental variable pairs

## Outputs

- Empirical p-values for each observed tau, derived from the 100-permutation null density
- Benjamini–Hochberg-adjusted p-values and significance flags for each taxon/environmental pair
- fdr_tau_results.csv containing taxon pair, tau, empirical p-value, adjusted p-value, and significance flag
- Set of significant pairs (corrected significance threshold tau = 0.70) feeding the community network analysis

## How to apply

Load the 18-month Monterey Bay multilocus amplicon abundance table and aligned environmental measurements (sea surface temperature, chlorophyll a, etc.) from github.com/marinebon/eDNA_microbes_whales, then compute all pairwise Kendall taus on proportional-abundance time-series with scipy.stats.kendalltau. Build an empirical null by permuting each abundance time-series (column-wise shuffling) 100 times and recomputing the full pairwise tau matrix per permutation with numpy. Locate each observed tau within the 100-permutation null density to derive an empirical p-value for every pair. Apply the Benjamini–Hochberg adjustment via statsmodels.stats.multitest.multipletests to the empirical p-values (equivalent to a Bonferroni-style multiple-testing correction at this scale), flagging pairs significant only where the corrected statistic clears the reported threshold of tau = 0.70. Rationale: with ~6.5×10^4 taxon/environmental pairs feeding a community network analysis, uncorrected p-values would yield many spurious predator–prey/trophic edges.

## Related tools

- **statsmodels** (Apply Benjamini–Hochberg false-discovery-rate adjustment to empirical p-values via statsmodels.stats.multitest.multipletests)
- **numpy** (Column-wise permutation of abundance time-series and recomputation of the pairwise tau matrix to build the empirical null density)
- **scipy** (Compute pairwise Kendall tau rank correlations on proportional-abundance time-series with scipy.stats.kendalltau)
- **pandas** (Load the 18-month abundance table and aligned environmental measurements from the eDNA_microbes_whales repository) — github.com/marinebon/eDNA_microbes_whales
- **networkx** (Downstream construction of the community network from FDR-significant tau pairs)

## Examples

```
from statsmodels.stats.multitest import multipletests; adj_p, flags, _, _ = multipletests(empirical_pvals, method='fdr_bh'); significant = [(pair, tau) for (pair, tau), f, t in zip(pairs, flags, taus) if f and t >= 0.70]
```

## Evaluation signals

- Reproduced corrected significance threshold equals tau = 0.70 after FDR adjustment of the empirical p-values.
- Number of significant pairs retained after Benjamini–Hochberg adjustment matches the community network reported in the paper.
- Null distribution is built from exactly 100 column-wise permutations of the abundance time-series, each yielding a full pairwise tau matrix.
- Adjusted p-values are monotonically consistent with empirical p-values under BH step-up ordering (no rank inversions).
- Every flagged-significant pair has adjusted p-value at or below the FDR-controlled level and tau ≥ 0.70.

## Limitations

- Whether co-occurrence reflects a direct predator–prey relationship or a common response to an external driver is unclear; FDR-significant edges are putative interactions, not confirmed interactions.
- Network connectivity does not necessarily indicate interaction, even for pairs surviving the tau = 0.70 corrected threshold.
- Amplification of DNA is subject to primer biases, inhibitors, and variable template abundances, which affect which taxa enter the correlation set at all; no method identifies all taxa in an ecosystem.
- The 100-permutation null limits empirical p-value resolution at the extreme tail; hypotheses from these network analyses should be further investigated through hypothesis-based research and experiments, with longer time-series needed to increase power and validity.

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Although network connectivity does not necessarily indicate interaction...: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis...: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [other] Generate a null distribution by permuting the time-series 100 times (column-wise shuffling of abundance vectors)...: [evidence span withheld — anchor status: not_found]
