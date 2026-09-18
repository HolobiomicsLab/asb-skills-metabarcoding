---
name: predator-prey-linkage-validation
description: Use when verifying that a reported taxon-pair co-occurrence correlation
  (e.g., Metridinidae-Carangidae r=0.80, p<0.05) from a seasonal eDNA subnetwork is
  reproducible from the stored correlation/adjacency matrix.
when_to_use_negative:
- Input is raw eDNA amplicon sequencing data or an abundance feature table rather
  than a precomputed subnetwork correlation/adjacency matrix — computing correlations
  is a separate upstream step.
- The goal is to discover or claim direct predator-prey interactions from co-occurrence
  alone; the article itself cautions that co-occurrence may reflect a common response
  to an external driver rather than predation.
- The taxon pair of interest does not appear in the grey (winter) subnetwork matrix
  (e.g., a pair from a different seasonal subnetwork).
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: pandas
  role: Loads the grey subnetwork's pairwise taxon correlation matrix into a DataFrame
    for taxon-pair value extraction
  repo: github.com/marinebon/eDNA_microbes_whales
- name: SciPy
  role: Provides the Spearman correlation coefficient and p-value fields compared
    against the reported r=0.80 and p<0.05 thresholds
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:648e6e219160d995d445b7cd8d1dca12cddb2a12a035d3b824074a80b5bd53d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# predator-prey-linkage-validation

## Summary

Validates putative predator-prey co-occurrence linkages (e.g., Metridinidae copepods-Carangidae jacks, r=0.80, p<0.05; Otariidae sea lions-Carangidae) reported from the grey (winter) subnetwork of a Monterey Bay eDNA community network analysis, by programmatically extracting Spearman correlation coefficients and p-values from the repository's pairwise taxon correlation matrix.

## When to use

Use when verifying that a reported taxon-pair co-occurrence correlation (e.g., Metridinidae-Carangidae r=0.80, p<0.05) from a seasonal eDNA subnetwork is reproducible from the stored correlation/adjacency matrix.

## When NOT to use

- Input is raw eDNA amplicon sequencing data or an abundance feature table rather than a precomputed subnetwork correlation/adjacency matrix — computing correlations is a separate upstream step.
- The goal is to discover or claim direct predator-prey interactions from co-occurrence alone; the article itself cautions that co-occurrence may reflect a common response to an external driver rather than predation.
- The taxon pair of interest does not appear in the grey (winter) subnetwork matrix (e.g., a pair from a different seasonal subnetwork).

## Inputs

- Pairwise taxon Spearman correlation matrix for the grey (winter) subnetwork (from github.com/marinebon/eDNA_microbes_whales)
- Network adjacency structure containing the Otariidae-Carangidae edge
- Reported reference values: Metridinidae-Carangidae r=0.80, Spearman p<0.05

## Outputs

- correlation_verification.csv with extracted r, extracted p, pass/fail vs r=0.80, pass/fail vs p<0.05, and Otariidae-Carangidae edge-confirmation status

## How to apply

Load the grey (winter) subnetwork's pairwise taxon correlation matrix from the github.com/marinebon/eDNA_microbes_whales repository into a pandas DataFrame. Extract the Spearman correlation coefficient and p-value stored for the Metridinidae-Carangidae taxon pair, and record pass/fail against the reported thresholds r=0.80 and p<0.05. Separately inspect the network adjacency for the Otariidae-Carangidae pair to confirm the reported predator-prey co-occurrence edge. Write the extracted values, comparison outcomes, and edge-confirmation status to correlation_verification.csv. Rationale: the paper's community network analysis of an 18-month Monterey Bay seawater time-series (663 taxonomic groups, Family or higher) is the basis for these reported trophic linkages, so validation must operate on the same subnetwork-level correlation artifacts rather than raw reads.

## Related tools

- **pandas** (Loads the grey subnetwork's pairwise taxon correlation matrix into a DataFrame for taxon-pair value extraction) — github.com/marinebon/eDNA_microbes_whales
- **SciPy** (Provides the Spearman correlation coefficient and p-value fields compared against the reported r=0.80 and p<0.05 thresholds) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; m = pd.read_csv('grey_subnetwork_correlations.csv', index_col=0); print(m.loc[['Metridinidae','Carangidae'], ['Carangidae','spearman_p']])
```

## Evaluation signals

- Extracted Metridinidae-Carangidae Spearman r reproduces the reported value of 0.80 within numerical tolerance.
- Extracted Metridinidae-Carangidae p-value satisfies the reported p<0.05 threshold.
- Pass/fail flags in correlation_verification.csv match the comparison outcomes for both thresholds.
- The Otariidae-Carangidae edge is confirmed present in the network adjacency, consistent with the reported co-occurrence linkage.
- correlation_verification.csv contains a complete row (r, p, pass/fail flags, edge status) with no missing values.

## Limitations

- Whether taxon co-occurrence is driven by direct predator-prey relationships or a common response to an external driver is unclear.
- Network connectivity does not necessarily indicate interaction, though a more connected taxon is more likely to present similar patterns of abundance.
- Amplification of DNA is subject to substantial primer biases and is affected by inhibitors and variable abundances of template DNA; eDNA may include sequences from dead or decaying biological matter and cannot identify life-history stages.
- No method, including this one, identifies all taxa in an ecosystem, so absence of a pair from the matrix may reflect detection limits.
- The repository README contains no documentation of the correlation matrix format or file locations, so exact artifact paths must be located in the repository.

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear.: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [discussion] Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
