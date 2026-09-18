---
name: reported-statistic-reproduction-verification
description: Use when a paper reports specific correlation coefficients between community
  subnetwork properties (e.g., taxon richness of WGCNA subnetworks) and environmental
  time-series (e.g., chlorophyll a, sea surface temperature) and the task is to independently
  reproduce those exact statistics.
when_to_use_negative:
- The upstream WGCNA subnetwork memberships have not yet been computed — compute the
  subnetworks first via the community-network-analysis skill before correlating richness
  with environmental variables.
- The goal is exploratory identification of new correlations rather than verification
  of pre-specified published statistics; then a full exploratory correlation matrix
  with multiple-testing correction is more appropriate.
- The environmental and richness tables are not sample-aligned (e.g., different sampling
  dates or missing SST/chlorophyll a records), which would invalidate the rank correlation.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: WGCNA
  role: Upstream source of the subnetwork (grey/blue module) membership and richness
    profiles whose correlation with environmental variables is being verified
  repo: github.com/marinebon/eDNA_microbes_whales
- name: scipy
  role: Provides scipy.stats.spearmanr for computing the Spearman rank correlation
    coefficients and p-values
- name: pandas
  role: Loads and aligns the subnetwork richness table and the SST/chlorophyll a time-series,
    and holds the results table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:89de1e8c168c32492f545fb6ef671d0073e279842b39117961f01e2841df1f3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - WGCNA
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Reported statistic reproduction verification

## Summary

Recompute the Monterey Bay eDNA subnetwork-environment Spearman correlations (grey vs chlorophyll a; blue vs sea surface temperature) from the WGCNA subnetwork membership and richness tables and verify the reproduced r and p values against the published statistics (r = -0.76, p = 0.03; r = 0.74, p = 0.04).

## When to use

Use when a paper reports specific correlation coefficients between community subnetwork properties (e.g., taxon richness of WGCNA subnetworks) and environmental time-series (e.g., chlorophyll a, sea surface temperature) and the task is to independently reproduce those exact statistics.

## When NOT to use

- The upstream WGCNA subnetwork memberships have not yet been computed — compute the subnetworks first via the community-network-analysis skill before correlating richness with environmental variables.
- The goal is exploratory identification of new correlations rather than verification of pre-specified published statistics; then a full exploratory correlation matrix with multiple-testing correction is more appropriate.
- The environmental and richness tables are not sample-aligned (e.g., different sampling dates or missing SST/chlorophyll a records), which would invalidate the rank correlation.

## Inputs

- WGCNA subnetwork membership and taxon-richness/member-profile table (chained from the community-network-analysis sub-task)
- Monterey Bay environmental variable time-series (sea surface temperature, chlorophyll a) aligned to the 18-month 2015-2016 eDNA seawater sampling dates

## Outputs

- Results table of Spearman correlation coefficients and p-values for every subnetwork-environment pair
- Validation report flagging pass/fail for the grey vs chlorophyll a (r = -0.76, p = 0.03) and blue vs sea surface temperature (r = 0.74, p = 0.04) rows

## How to apply

Load the WGCNA subnetwork membership and richness/member-profile table chained from the upstream community-network-analysis sub-task alongside the Monterey Bay 2015-2016 seawater time-series of environmental variables (sea surface temperature, chlorophyll a). Compute Spearman rank correlations between each subnetwork richness/member profile and each environmental variable with scipy.stats.spearmanr (or R cor.test with method='spearman'). Record the correlation coefficient and p-value for every subnetwork-environment pair in a results table before filtering. Extract the grey-subnetwork vs chlorophyll a row and the blue-subnetwork vs sea surface temperature row, then compare the recomputed r and p values against the reported statistics. Accept the output only if the recomputed grey vs chlorophyll a Spearman r = -0.76 (p = 0.03) and blue vs SST r = 0.74 (p = 0.04) match the reported values; this validation step guards against upstream drift in the network construction or environmental table.

## Related tools

- **WGCNA** (Upstream source of the subnetwork (grey/blue module) membership and richness profiles whose correlation with environmental variables is being verified) — github.com/marinebon/eDNA_microbes_whales
- **scipy** (Provides scipy.stats.spearmanr for computing the Spearman rank correlation coefficients and p-values)
- **pandas** (Loads and aligns the subnetwork richness table and the SST/chlorophyll a time-series, and holds the results table)

## Examples

```
import pandas as pd; from scipy.stats import spearmanr; df = pd.merge(richness, env, on='sample_date'); r, p = spearmanr(df['grey_richness'], df['chlorophyll_a'])
```

## Evaluation signals

- Recomputed Spearman r for grey-subnetwork richness vs chlorophyll a equals -0.76 with p = 0.03
- Recomputed Spearman r for blue-subnetwork richness vs sea surface temperature equals 0.74 with p = 0.04
- Results table contains one row for every subnetwork-environment pair tested (full matrix, not just the two reported pairs)
- All correlations use Spearman rank method (scipy.stats.spearmanr or R cor.test method='spearman'), not Pearson
- Sample alignment between richness values and environmental measurements is complete before correlation is computed

## Limitations

- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear, so significant subnetwork-environment correlations do not establish mechanism
- Amplification of DNA is subject to primer biases and is affected by inhibitors and variable template abundances, which propagate into the richness values being correlated
- The article notes that longer time-series are needed to increase power and validity of these eDNA-based analyses

## Evidence

- [intro] objective association quote: "This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a, and other key"
- [discussion] subnetworks fluctuate over time: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [abstract] network analysis provided evidence: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [readme] repo purpose: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
