---
name: marine-taxon-pair-identification
description: Use when a pairwise taxon co-occurrence claim from a community network
  analysis (e.g., Metridinidae-Carangidae, Otariidae-Carangidae) must be quantitatively
  verified against a stored correlation matrix with r and p-value fields.
when_to_use_negative:
- When the goal is to prove direct predation — the paper states co-occurrence may
  be 'a common response to an external driver' and 'network connectivity does not
  necessarily indicate interaction'
- When the input is raw sequencing data or an abundance feature table rather than
  a precomputed pairwise correlation matrix / network adjacency
- When the taxon pair of interest is absent from the grey (winter) subnetwork's stored
  correlation matrix or adjacency list
edam_operation: http://edamontology.org/operation_2945
tools:
- name: pandas
  role: Load and index the grey subnetwork's pairwise taxon correlation matrix (e.g.,
    extracting the Metridinidae-Carangidae cell)
  repo: github.com/marinebon/eDNA_microbes_whales
- name: SciPy
  role: Access Spearman correlation coefficient and p-value fields stored in the matrix
    and support the p < 0.05 threshold comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:dbf679427f7fd3735796105ce56b9f5ad25e2157ab36f3af05a679bde8266541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# marine-taxon-pair-identification

## Summary

Verifies putative predator-prey co-occurrence linkages between marine taxa by extracting Spearman correlation coefficients and p-values for specific taxon pairs (e.g., Metridinidae copepods vs. Carangidae jacks) from a community network's subnetwork correlation matrix, and comparing them against reported thresholds such as r = 0.80 and p < 0.05.

## When to use

Use when a pairwise taxon co-occurrence claim from a community network analysis (e.g., Metridinidae-Carangidae, Otariidae-Carangidae) must be quantitatively verified against a stored correlation matrix with r and p-value fields.

## When NOT to use

- When the goal is to prove direct predation — the paper states co-occurrence may be 'a common response to an external driver' and 'network connectivity does not necessarily indicate interaction'
- When the input is raw sequencing data or an abundance feature table rather than a precomputed pairwise correlation matrix / network adjacency
- When the taxon pair of interest is absent from the grey (winter) subnetwork's stored correlation matrix or adjacency list

## Inputs

- Pairwise Spearman correlation matrix for the grey (winter) subnetwork taxa (from the github.com/marinebon/eDNA_microbes_whales repository)
- Network adjacency structure for the subnetwork containing the taxon pairs of interest
- Reported reference thresholds for the taxon pair (r = 0.80, p < 0.05)

## Outputs

- correlation_verification.csv containing extracted Spearman r and p values for the Metridinidae-Carangidae pair, pass/fail comparison outcomes, and Otariidae-Carangidae edge-confirmation status
- A summary judgment of whether the winter subnetwork quantitatively supports the reported trophic chain (sea lions prey upon jacks that feed upon planktonic copepods)

## How to apply

Load the grey (winter) subnetwork's pairwise taxon Spearman correlation matrix from the github.com/marinebon/eDNA_microbes_whales repository using pandas. Extract the correlation coefficient and p-value for the target taxon pair (here Metridinidae vs. Carangidae) from the matrix fields. Compare the extracted values against the reported thresholds (r = 0.80, p < 0.05) and record pass/fail for each. Separately inspect the network adjacency for the predator-prey edge (Otariidae-Carangidae) to confirm the reported co-occurrence linkage. Write extracted values, comparison outcomes, and edge-confirmation status to a CSV (correlation_verification.csv) so the verification is auditable. Note that the paper itself cautions that co-occurrence may reflect direct predation or a shared response to an external driver, so frame conclusions as putative trophic linkage support, not proof of interaction.

## Related tools

- **pandas** (Load and index the grey subnetwork's pairwise taxon correlation matrix (e.g., extracting the Metridinidae-Carangidae cell)) — github.com/marinebon/eDNA_microbes_whales
- **SciPy** (Access Spearman correlation coefficient and p-value fields stored in the matrix and support the p < 0.05 threshold comparison)

## Examples

```
import pandas as pd; from scipy.stats import spearmanr; corr = pd.read_csv('grey_subnetwork_correlations.csv', index_col=0); r = corr.loc['Metridinidae', 'Carangidae']
```

## Evaluation signals

- Extracted Spearman coefficient for Metridinidae-Carangidae matches the reported r = 0.80 within numerical tolerance
- Extracted p-value satisfies the reported threshold p < 0.05, with the pass/fail outcome explicitly recorded
- correlation_verification.csv exists and contains the extracted values, comparison outcomes, and edge-confirmation status fields
- The Otariidae-Carangidae (predator-prey) edge is positively confirmed in the network adjacency as a co-occurrence linkage
- The analysis is restricted to the grey (winter) subnetwork rather than pooled or other seasonal subnetworks

## Limitations

- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Network connectivity does not necessarily indicate interaction, although a more connected taxon is more likely to show similar abundance patterns
- Hypotheses from network analyses require further hypothesis-based research and experiments; longer time-series are needed to increase power and validity

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
