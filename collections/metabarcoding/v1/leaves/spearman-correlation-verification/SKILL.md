---
name: spearman-correlation-verification
description: Use when a published eDNA community-network claim about taxon co-occurrence
  (e.g. predator–prey pairs like Metridinidae–Carangidae or Otariidae–Carangidae)
  must be verified against an available pairwise correlation matrix before relying
  on the linkage.
when_to_use_negative:
- The input is raw amplicon sequencing data or a taxonomic abundance table rather
  than a precomputed pairwise correlation/adjacency matrix — this skill verifies reported
  correlations, it does not compute them de novo.
- The claimed linkage involves a seasonal subnetwork other than the grey (winter)
  subnetwork; the reported Metridinidae–Carangidae r = 0.80 value is specific to the
  grey subnetwork.
- The goal is to establish a causal predator–prey relationship; the article notes
  co-occurrence may reflect a common response to an external driver and that network
  analyses require further hypothesis-based research and experiments.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0610
tools:
- name: pandas
  role: Loads the grey subnetwork's pairwise taxon correlation matrix and extracts
    the Metridinidae–Carangidae row/column values
  repo: github.com/marinebon/eDNA_microbes_whales
- name: SciPy
  role: Provides the Spearman correlation coefficient and p-value fields (scipy.stats-supported)
    against which the reported r = 0.80 and p < 0.05 thresholds are compared
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:06aaedb339596023f5a5fe3ea51aee93b12294ba77cb5278c5c33d27e154262a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 8
---

# spearman-correlation-verification

## Summary

Reproduces reported taxon co-occurrence Spearman correlations (e.g. Metridinidae–Carangidae r = 0.80, p < 0.05) from the Monterey Bay eDNA grey (winter) subnetwork, verifying quantitatively that putative predator–prey trophic linkages are supported by the published correlation matrix.

## When to use

Use when a published eDNA community-network claim about taxon co-occurrence (e.g. predator–prey pairs like Metridinidae–Carangidae or Otariidae–Carangidae) must be verified against an available pairwise correlation matrix before relying on the linkage.

## When NOT to use

- The input is raw amplicon sequencing data or a taxonomic abundance table rather than a precomputed pairwise correlation/adjacency matrix — this skill verifies reported correlations, it does not compute them de novo.
- The claimed linkage involves a seasonal subnetwork other than the grey (winter) subnetwork; the reported Metridinidae–Carangidae r = 0.80 value is specific to the grey subnetwork.
- The goal is to establish a causal predator–prey relationship; the article notes co-occurrence may reflect a common response to an external driver and that network analyses require further hypothesis-based research and experiments.

## Inputs

- Grey (winter) subnetwork pairwise taxon Spearman correlation matrix (with coefficient and p-value fields) from github.com/marinebon/eDNA_microbes_whales
- Network adjacency/edge list for the grey subnetwork
- Target taxon pair identifiers (e.g. Metridinidae, Carangidae, Otariidae)

## Outputs

- correlation_verification.csv with extracted r, extracted p, r-threshold pass/fail (r = 0.80), p-threshold pass/fail (p < 0.05), and Otariidae–Carangidae edge-confirmation status

## How to apply

Load the grey (winter) subnetwork's pairwise taxon Spearman correlation matrix from the marinebon/eDNA_microbes_whales repository using pandas, since the network encodes seasonal subnetworks whose co-occurrence edges support trophic-linkage claims. Extract the Spearman correlation coefficient and p-value for the target taxon pair (Metridinidae–Carangidae) from the matrix fields. Compare the extracted values against the reported thresholds r = 0.80 and p < 0.05, recording a pass/fail outcome for each threshold. Separately inspect the network adjacency for the Otariidae–Carangidae pair to confirm the reported co-occurrence (predator–prey) edge, consistent with the trophic chain in which California sea lions prey upon jacks that feed upon planktonic copepods. Write the extracted coefficient, p-value, threshold comparisons, and edge-confirmation status to correlation_verification.csv. Interpret any confirmed edge cautiously: the paper states network connectivity does not necessarily indicate a direct interaction.

## Related tools

- **pandas** (Loads the grey subnetwork's pairwise taxon correlation matrix and extracts the Metridinidae–Carangidae row/column values) — github.com/marinebon/eDNA_microbes_whales
- **SciPy** (Provides the Spearman correlation coefficient and p-value fields (scipy.stats-supported) against which the reported r = 0.80 and p < 0.05 thresholds are compared) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; m = pd.read_csv('grey_subnetwork_correlations.csv'); r, p = m.loc['Metridinidae', 'Carangidae']; print(r, p < 0.05, r == 0.80)
```

## Evaluation signals

- Extracted Spearman coefficient for Metridinidae–Carangidae equals the reported r = 0.80 (allowing float tolerance).
- Extracted p-value satisfies p < 0.05; a fail here means the reported significance is not reproduced from the grey subnetwork matrix.
- The Otariidae–Carangidae pair is present as an edge in the grey subnetwork adjacency, confirming the reported predator–prey co-occurrence linkage.
- correlation_verification.csv contains all required fields: extracted r, extracted p, pass/fail per threshold, and edge-confirmation status, with no missing taxon-pair lookups (NaN).

## Limitations

- Whether taxon co-occurrence is driven by direct predator–prey relationships or a common response to an external driver is unclear, so a verified edge supports a putative, not proven, interaction.
- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to show similar abundance patterns.
- eDNA cannot identify life history stages and may include sequences from dead or decaying biological matter, and amplification is subject to primer biases and inhibitors, which can affect taxon detection and thus correlations.
- Hypotheses from network analyses require further hypothesis-based research and experiments; the repository README contains no additional pipeline documentation beyond a statement of purpose for the eDNA group.

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life"
- [discussion] Network connectivity caveat: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [discussion] Co-occurrence driver ambiguity: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [readme] Repository README: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
