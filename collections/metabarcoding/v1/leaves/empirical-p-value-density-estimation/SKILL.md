---
name: empirical-p-value-density-estimation
description: Use when a correlation matrix over many taxa × environmental variable
  pairs from a eDNA time-series must be tested for significance, and the multiple-comparison
  burden requires an empirical permutation null rather than parametric p-values.
when_to_use_negative:
- When testing a single or very small number of pre-specified associations, where
  no multiple-comparison correction or permutation null density is needed.
- When the input is raw sequencing reads or a quality-filtering problem (the repository
  README scopes the repo to sequencing data analysis, quality filtering, and bioinformatics);
  this skill operates downstream on the proportional-abundance table.
- When taxon detection counts must be interpreted absolutely — the article notes primer
  biases, inhibitors, and variable template abundances affect which taxa are detected,
  so tau thresholds should not be read as absolute abundance relationships.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3307
tools:
- name: scipy.stats.kendalltau
  role: computes the pairwise Kendall tau rank correlations on the proportional-abundance
    time-series
- name: numpy
  role: performs the column-wise shuffling/permutation of abundance vectors and recomputation
    of taus to build the null density
- name: statsmodels.stats.multitest.multipletests
  role: applies the Benjamani-Hochberg false-discovery-rate adjustment to the empirical
    p-values
- name: pandas
  role: loads the eDNA abundance table and aligned environmental-property measurements
    from the repository
  repo: github.com/marinebon/eDNA_microbes_whales
- name: networkx
  role: consumes the significant-pair list for the community-network analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:d614585b491567173379a5ad6a24ac1d76058bfd601bb4c42646ec32f7820903
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
  - scipy.stats.kendalltau
  - statsmodels
  - statsmodels.stats.multitest.multipletests
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# empirical-p-value-density-estimation

## Summary

Builds a permutation null density of pairwise Kendall tau correlations over an 18-month Monterey Bay eDNA proportional-abundance time-series (663 taxonomic groups plus environmental variables), then derives empirical p-values and a Benjamani-Hochberg false-discovery-rate correction that yields a corrected significance threshold of tau = 0.70.

## When to use

Use when a correlation matrix over many taxa × environmental variable pairs from a eDNA time-series must be tested for significance, and the multiple-comparison burden requires an empirical permutation null rather than parametric p-values.

## When NOT to use

- When testing a single or very small number of pre-specified associations, where no multiple-comparison correction or permutation null density is needed.
- When the input is raw sequencing reads or a quality-filtering problem (the repository README scopes the repo to sequencing data analysis, quality filtering, and bioinformatics); this skill operates downstream on the proportional-abundance table.
- When taxon detection counts must be interpreted absolutely — the article notes primer biases, inhibitors, and variable template abundances affect which taxa are detected, so tau thresholds should not be read as absolute abundance relationships.

## Inputs

- 18-month (2015-2016) Monterey Bay eDNA multilocus amplicon abundance table (663 taxonomic groups, Family or higher) from github.com/marinebon/eDNA_microbes_whales
- Aligned environmental-property measurements (e.g., sea surface temperature, chlorophyll a) over the same sampling dates
- Proportional-abundance time-series derived from the raw abundance table

## Outputs

- fdr_tau_results.csv with columns: taxon pair, Kendall tau, empirical p-value, BH-adjusted p-value, significance flag
- Empirical null density of pairwise Kendall tau values from 100 permutations
- Corrected significance threshold (tau = 0.70) and list of significant taxon/environmental pairs feeding the community-network analysis

## How to apply

Load the 18-month Monterey Bay seawater eDNA multilocus amplicon abundance table together with aligned environmental properties (sea surface temperature, chlorophyll a, etc.) from github.com/marinebon/eDNA_microbes_whales using pandas, and convert to proportional-abundance time-series. Compute all pairwise Kendall tau rank correlations across the ~6.5x10^4 taxon/environmental variable pairs with scipy.stats.kendalltau. Build an empirical null distribution by column-wise shuffling of the abundance vectors 100 times and recomputing the full pairwise Kendall tau set per permutation with numpy, then locate each observed tau within this 100-permutation null density to obtain empirical p-values. Apply the Benjamani-Hochberg false-discovery-rate adjustment (statsmodels.stats.multitest.multipletests) and flag pairs as significant only where the corrected statistic clears the reported corrected threshold of tau = 0.70. Write per-pair results (taxon pair, tau, empirical p-value, adjusted p-value, significance flag) to fdr_tau_results.csv for downstream community-network analysis.

## Related tools

- **scipy.stats.kendalltau** (computes the pairwise Kendall tau rank correlations on the proportional-abundance time-series)
- **numpy** (performs the column-wise shuffling/permutation of abundance vectors and recomputation of taus to build the null density)
- **statsmodels.stats.multitest.multipletests** (applies the Benjamani-Hochberg false-discovery-rate adjustment to the empirical p-values)
- **pandas** (loads the eDNA abundance table and aligned environmental-property measurements from the repository) — github.com/marinebon/eDNA_microbes_whales
- **networkx** (consumes the significant-pair list for the community-network analysis)

## Examples

```
from scipy.stats import kendalltau; from statsmodels.stats.multitest import multipletests; tau, p = kendalltau(taxon_abundances, env_series)
```

## Evaluation signals

- Reproduced corrected significance threshold equals tau = 0.70.
- Number of significant pairs retained after FDR adjustment matches the community-network analysis reported in the paper.
- Null distribution is built from exactly 100 column-wise permutations of the abundance time-series.
- Every reported empirical p-value falls within [0,1] and is consistent with the rank of the observed tau in the permutation null density.
- Output CSV schema matches (taxon pair, tau, empirical p-value, adjusted p-value, significance flag) with no missing pairs relative to the full pairwise grid.

## Limitations

- Network connectivity does not necessarily indicate interaction; co-occurrence may reflect a common response to an external driver rather than a direct predator-prey relationship.
- Primer biases, inhibitors, and variable template DNA abundances affect taxon detection, so the correlation graph is conditional on the taxa the method detected (no method identifies all taxa).
- Hypotheses generated from the network analyses require further hypothesis-based research and experiments; longer time-series are needed to increase power and validity.

## Evidence

- [abstract] The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals.: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [intro] Here, we analyse a time-series of eDNA metabarcoding data to assess changes in the proportional abundance of taxa in a marine setting: "Here, we analyse a time-series of eDNA metabarcoding data to assess changes in the proportional abundance of taxa in a marine setting"
- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
