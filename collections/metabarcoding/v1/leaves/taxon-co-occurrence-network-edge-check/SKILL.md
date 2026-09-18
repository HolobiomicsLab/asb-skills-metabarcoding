---
name: taxon-co-occurrence-network-edge-check
description: Use when a claimed predator-prey or trophic edge in a co-occurrence network
  (e.g., Otariidae-Carangidae, Metridinidae-Carangidae) must be reproduced from a
  stored correlation matrix before citing it.
when_to_use_negative:
- The goal is to infer new co-occurrence edges from raw eDNA abundance data rather
  than verify edges already reported in a published network
- The taxa of interest are not present in the subnetwork correlation matrix or adjacency
  structure (the correlation matrix must contain the target pair's fields)
- The question is whether a co-occurrence reflects a direct predator-prey relationship
  versus a shared environmental driver — the article states this distinction cannot
  be resolved by network connectivity alone
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: pandas
  role: Loads the grey subnetwork's pairwise taxon correlation matrix into a DataFrame
    for extracting the Metridinidae-Carangidae coefficient and p-value
  repo: github.com/marinebon/eDNA_microbes_whales
- name: SciPy
  role: Source of the Spearman correlation statistics (coefficient and p-value) stored
    in the matrix fields being verified
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:1279997b871576da9477670550afe7507b459c24d667c23c788abd38653d8a83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# taxon-co-occurrence-network-edge-check

## Summary

Verifies reported taxon co-occurrence edges in an eDNA community network (e.g., Metridinidae–Carangidae r=0.80, p<0.05 in the grey winter subnetwork of the Monterey Bay 18-month time-series) by extracting Spearman coefficients and p-values from the pairwise correlation matrix and confirming adjacency edges such as Otariidae–Carangidae.

## When to use

Use when a claimed predator-prey or trophic edge in a co-occurrence network (e.g., Otariidae-Carangidae, Metridinidae-Carangidae) must be reproduced from a stored correlation matrix before citing it.

## When NOT to use

- The goal is to infer new co-occurrence edges from raw eDNA abundance data rather than verify edges already reported in a published network
- The taxa of interest are not present in the subnetwork correlation matrix or adjacency structure (the correlation matrix must contain the target pair's fields)
- The question is whether a co-occurrence reflects a direct predator-prey relationship versus a shared environmental driver — the article states this distinction cannot be resolved by network connectivity alone

## Inputs

- Grey (winter) subnetwork pairwise taxon Spearman correlation matrix (with scipy.stats coefficient and p-value fields) from the marinebon/eDNA_microbes_whales repository
- Network adjacency structure listing taxon-taxone co-occurrence edges
- Reported threshold values (r=0.80, p<0.05) for the Metridinidae-Carangidae pair

## Outputs

- correlation_verification.csv containing extracted r and p values, pass/fail comparison outcomes, and Otariidae-Carangidae edge-confirmation status

## How to apply

Load the grey (winter) subnetwork's pairwise taxon correlation matrix from the github.com/marinebon/eDNA_microbes_whales repository into a pandas DataFrame. Extract the Spearman correlation coefficient and p-value for the target taxon pair (e.g., Metridinidae-Carangidae) from the scipy.stats-supported fields stored in the matrix. Apply the reported thresholds as pass/fail criteria: r = 0.80 and Spearman p < 0.05. Separately inspect the network adjacency for the second reported predator-prey pair (Otariidae-Carangidae) to confirm the edge exists. Write extracted values, pass/fail outcomes, and edge-confirmation status to correlation_verification.csv.

## Related tools

- **pandas** (Loads the grey subnetwork's pairwise taxon correlation matrix into a DataFrame for extracting the Metridinidae-Carangidae coefficient and p-value) — github.com/marinebon/eDNA_microbes_whales
- **SciPy** (Source of the Spearman correlation statistics (coefficient and p-value) stored in the matrix fields being verified) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; m = pd.read_csv('grey_subnetwork_correlations.csv'); r, p = m.loc['Metridinidae','Carangidae_r'], m.loc['Metridinidae','Carangidae_p']; print(r, p < 0.05)
```

## Evaluation signals

- Extracted Metridinidae-Carangidae Spearman r equals the reported 0.80 (within numerical tolerance of the stored matrix)
- Extracted Metridinidae-Carangidae p-value satisfies p < 0.05, matching the reported significance
- The Otariidae-Carangidae edge is present in the network adjacency, confirming the reported predator-prey co-occurrence linkage
- correlation_verification.csv is produced with one row per check containing extracted values, pass/fail outcomes, and edge-confirmation status
- Values are read from scipy.stats-supported fields in the matrix rather than recomputed or assumed

## Limitations

- Network connectivity does not necessarily indicate interaction; co-occurrence may be driven by direct predator-prey relationships or a common response to an external driver
- eDNA cannot identify life history stages and may include sequences from dead or decaying biological matter, affecting the biological interpretation of the edge
- Amplification is subject to primer biases, inhibitors, and variable template DNA abundances, which affect taxon detection underlying the correlations
- Taxon interdependencies suggested by network analyses should be further investigated through hypothesis-based research and experiments

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life"
- [discussion] whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear.: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
