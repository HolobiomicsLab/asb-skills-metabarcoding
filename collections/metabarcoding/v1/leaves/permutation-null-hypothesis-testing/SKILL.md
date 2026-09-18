---
name: permutation-null-hypothesis-testing
description: Use when many pairwise rank correlations (e.g., taxon abundance or taxon-vs-environment
  pairs) must be tested on a short ecological time-series and raw per-test p-values
  would inflate false discoveries.
when_to_use_negative:
- The data are already an interaction network or feature table with precomputed significance
  calls — rerunning the null would double-correct.
- Only a handful of correlations are being tested, where a permutation null and FDR
  correction at the tau = 0.70 scale are unnecessary.
- The abundance table is not a time-series (no temporal ordering), since column-wise
  shuffling of time-series is what generates the null.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scipy
  role: scipy.stats.kendalltau computes the pairwise Kendall tau rank correlations
    on proportional-abundance time-series
  repo: github.com/marinebon/eDNA_microbes_whales
- name: numpy
  role: column-wise permutation shuffling of abundance vectors and recomputation of
    pairwise taus to build the null density
  repo: github.com/marinebon/eDNA_microbes_whales
- name: statsmodels
  role: statsmodels.stats.multitest.multipletests applies the Benjamani-Hochberg false-discovery-rate
    adjustment to empirical p-values
  repo: github.com/marinebon/eDNA_microbes_whales
- name: pandas
  role: loads the 18-month abundance table and aligned environmental-property measurements
    from the repository
  repo: github.com/marinebon/eDNA_microbes_whales
- name: networkx
  role: consumes the significant tau pairs retained after FDR adjustment to build
    the community network
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:a57ac8717c5e44c48f5b311ba704d9a5ba45fdda81afdab243fe6449b24ec7cd
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
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# permutation-null-hypothesis-testing

## Summary

Controls the false-discovery rate for thousands of pairwise Kendall's tau correlations computed on the 18-month Monterey Bay eDNA time-series by building an empirical permutation null and reporting a corrected significance threshold of tau = 0.70.

## When to use

Use when many pairwise rank correlations (e.g., taxon abundance or taxon-vs-environment pairs) must be tested on a short ecological time-series and raw per-test p-values would inflate false discoveries.

## When NOT to use

- The data are already an interaction network or feature table with precomputed significance calls — rerunning the null would double-correct.
- Only a handful of correlations are being tested, where a permutation null and FDR correction at the tau = 0.70 scale are unnecessary.
- The abundance table is not a time-series (no temporal ordering), since column-wise shuffling of time-series is what generates the null.

## Inputs

- 18-month Monterey Bay eDNA multilocus amplicon proportional-abundance table (663 taxonomic groups, Family or higher)
- Aligned environmental-property time-series (sea surface temperature, chlorophyll a, other key environmental and biotic variables)
- Permutation count (100 column-wise shuffles of abundance vectors)

## Outputs

- Pairwise Kendall tau correlation matrix over taxon/environmental variable pairs
- Empirical p-values for each observed tau from the 100-permutation null density
- FDR-adjusted p-values with significance flags (threshold tau = 0.70) written to fdr_tau_results.csv
- Significant-pair list feeding the community network analysis (networkx)

## How to apply

Load the 18-month Monterey Bay eDNA multilocus amplicon proportional-abundance table and aligned environmental measurements (sea surface temperature, chlorophyll a, etc.) from github.com/marinebon/eDNA_microbes_whales with pandas. Compute all pairwise Kendall tau rank correlations over the ~6.5x10^4 taxon/environmental variable pairs using scipy.stats.kendalltau. Build an empirical null by permuting each abundance time-series column-wise 100 times and recomputing the full pairwise tau matrix per permutation with numpy, then locate each observed tau within the null density to derive empirical p-values. Apply the Benjamini–Hochberg FDR adjustment (Bonferroni-style multiple-testing correction at this scale) via statsmodels.stats.multitest.multipletests, flagging pairs as significant only where the corrected statistic clears the threshold tau = 0.70. Write per-pair results (taxon pair, tau, empirical p-value, adjusted p-value, significance flag) to fdr_tau_results.csv for downstream community network analysis.

## Related tools

- **scipy** (scipy.stats.kendalltau computes the pairwise Kendall tau rank correlations on proportional-abundance time-series) — github.com/marinebon/eDNA_microbes_whales
- **numpy** (column-wise permutation shuffling of abundance vectors and recomputation of pairwise taus to build the null density) — github.com/marinebon/eDNA_microbes_whales
- **statsmodels** (statsmodels.stats.multitest.multipletests applies the Benjamani-Hochberg false-discovery-rate adjustment to empirical p-values) — github.com/marinebon/eDNA_microbes_whales
- **pandas** (loads the 18-month abundance table and aligned environmental-property measurements from the repository) — github.com/marinebon/eDNA_microbes_whales
- **networkx** (consumes the significant tau pairs retained after FDR adjustment to build the community network) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
from scipy.stats import kendalltau; from statsmodels.stats.multitest import multipletests; tau, _ = kendalltau(x, y); p_emp = null_density_lookup(tau); sig = multipletests(p_emp, method='fdr_bh')[0]
```

## Evaluation signals

- The reproduced corrected significance threshold equals tau = 0.70.
- The number of significant pairs retained after Benjamini–Hochberg adjustment matches the community-network analysis reported in the paper.
- Empirical p-values fall within the resolution bounds of the 100-permutation null (i.e., multiples of 1/100) and adjusted p-values never exceed their raw empirical p-values.
- The output fdr_tau_results.csv contains the required columns: taxon pair, tau, empirical p-value, adjusted p-value, significance flag.
- The number of tested pairs is consistent with the ~6.5x10^4 taxon/environmental variable pairs implied by the 663-taxonomic-group table plus environmental variables.

## Limitations

- The null distribution resolution is limited by 100 permutations, so minimum empirical p-values are bounded at ~0.01.
- Network connectivity does not necessarily indicate interaction, and whether co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear.
- Amplification is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, which affects the abundance indices fed into the correlations.
- No method, including this one, identifies all taxa in an ecosystem, so the correlation universe is incomplete by construction.

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [abstract] We inferred changes in the composition of communities, revealing putative interactions among taxa and identifying correlations between these communities and environmental properties over time.: "We inferred changes in the composition of communities, revealing putative interactions among taxa and identifying correlations between these communities and environmental properties over time."
- [intro] This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a: "This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
