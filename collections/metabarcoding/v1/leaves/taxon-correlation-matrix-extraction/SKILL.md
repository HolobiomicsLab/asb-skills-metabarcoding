---
name: taxon-correlation-matrix-extraction
description: Use when a pairwise taxon correlation matrix from an eDNA community network
  subnetwork is available and specific taxon-pair co-occurrence values (e.g., r=0.80,
  p<0.05) must be extracted and verified against reported thresholds.
when_to_use_negative:
- When the goal is to establish direct predator-prey interactions rather than quantify
  co-occurrence — the article notes co-occurrence may reflect 'a common response to
  an external driver'
- When no seasonally partitioned (e.g., grey/winter) subnetwork correlation matrix
  exists for the taxon pairs of interest
- When taxon-pair values are needed at species level — the article flags that lower
  taxonomic assignment levels carry lower confidence
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3050
tools:
- name: pandas
  role: Loads and indexes the grey subnetwork pairwise taxon correlation matrix for
    extraction of the Metridinidae-Carangidae and Otariidae-Carangidae entries
  repo: github.com/marinebon/eDNA_microbes_whales
- name: SciPy
  role: Provides the Spearman correlation statistics (coefficient and p-value) stored
    in the matrix fields being extracted and compared
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:03d894147d08b7417266c0ccfbe6ee8fb225f7f1c9748d0897d83e85f99a24e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# taxon-correlation-matrix-extraction

## Summary

Extracts and verifies pairwise Spearman correlation coefficients and p-values between taxa (e.g., Metridinidae-Carangidae, Otariidae-Carangidae) from a seasonally partitioned eDNA community network correlation matrix, to quantitatively test reported predator-prey or trophic linkages.

## When to use

Use when a pairwise taxon correlation matrix from an eDNA community network subnetwork is available and specific taxon-pair co-occurrence values (e.g., r=0.80, p<0.05) must be extracted and verified against reported thresholds.

## When NOT to use

- When the goal is to establish direct predator-prey interactions rather than quantify co-occurrence — the article notes co-occurrence may reflect 'a common response to an external driver'
- When no seasonally partitioned (e.g., grey/winter) subnetwork correlation matrix exists for the taxon pairs of interest
- When taxon-pair values are needed at species level — the article flags that lower taxonomic assignment levels carry lower confidence

## Inputs

- Pairwise taxon Spearman correlation matrix from the grey (winter) subnetwork (pandas-readable CSV/table from github.com/marinebon/eDNA_microbes_whales)
- Reported reference values for the target taxon pair (r=0.80, p<0.05)
- Network adjacency structure containing the Otariidae-Carangidae edge

## Outputs

- correlation_verification.csv with extracted Spearman r and p-values for Metridinidae-Carangidae
- Pass/fail comparison outcomes against r=0.80 and p<0.05 thresholds
- Edge-confirmation status for the Otariidae-Carangidae co-occurrence linkage

## How to apply

Load the grey (winter) subnetwork's pairwise taxon Spearman correlation matrix from the github.com/marinebon/eDNA_microbes_whales repository into pandas as an indexed dataframe. Extract the correlation coefficient and p-value fields for the target taxon pair (Metridinidae-Carangidae) and compare against the reported thresholds r=0.80 and p<0.05, recording pass/fail for each. Separately inspect the network adjacency for the Otariidae-Carangidae pair to confirm the reported co-occurrence (predator-prey) edge. Rationale: the article's community network analysis of the 18-month Monterey Bay eDNA time-series clustered marine organisms into subnetworks whose co-occurrence patterns putatively reflect trophic linkages, so matrix-level verification checks whether the quantitative values support the qualitative claims. Write extracted values, comparison outcomes, and edge-confirmation status to an output CSV (e.g., correlation_verification.csv).

## Related tools

- **pandas** (Loads and indexes the grey subnetwork pairwise taxon correlation matrix for extraction of the Metridinidae-Carangidae and Otariidae-Carangidae entries) — github.com/marinebon/eDNA_microbes_whales
- **SciPy** (Provides the Spearman correlation statistics (coefficient and p-value) stored in the matrix fields being extracted and compared)

## Examples

```
import pandas as pd; m = pd.read_csv('grey_subnetwork_spearman_correlations.csv', index_col=0); r, p = m.loc['Metridinidae', 'Carangidae', 'r'], m.loc['Metridinidae', 'Carangidae', 'p']
```

## Evaluation signals

- Extracted Metridinidae-Carangidae Spearman r equals the reported 0.80
- Extracted p-value satisfies the reported threshold of p < 0.05
- Otariidae-Carangidae edge is present in the network adjacency, confirming the reported co-occurrence linkage
- correlation_verification.csv contains extracted values, pass/fail outcomes, and edge-confirmation status for all requested pairs
- Correlation values are drawn from the grey (winter) subnetwork matrix specifically, not a pooled or different-season matrix

## Limitations

- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Network connectivity does not necessarily indicate interaction; a more connected taxon is more likely to present similar abundance patterns
- Hypotheses from network analyses require further hypothesis-based research and experiments
- eDNA does not allow identification of life history stages and could include sequences from dead or decaying biological matter
- Amplification of DNA is subject to primer biases, inhibitors, and variable template abundances affecting taxa detection

## Evidence

- [results] copepods (especially Metridinidae) were strongly correlated to jacks (Carangidae), with r = 0.80 and a Spearman correlation p value < 0.05: [evidence span withheld — anchor status: not_found]
- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear.: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [discussion] Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time.: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
