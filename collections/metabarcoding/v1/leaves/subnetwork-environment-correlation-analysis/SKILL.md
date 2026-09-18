---
name: subnetwork-environment-correlation-analysis
description: Use when a community network analysis has produced subnetwork (module)
  memberships with richness or abundance profiles and you need to test their association
  with measured environmental variables such as chlorophyll a or sea surface temperature.
when_to_use_negative:
- Input is raw eDNA sequence data or an unclustered abundance feature table — subnetwork
  memberships must already exist from the upstream WGCNA community-network-analysis
  step
- You need to infer causal or direct predator-prey interactions from co-occurrence
  — the article notes it is unclear whether co-occurrence reflects direct interactions
  or a common response to an external driver
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: WGCNA
  role: Upstream producer of the subnetwork memberships and richness/member-profile
    tables that are correlated against environmental variables
- name: scipy
  role: scipy.stats.spearmanr computes the Spearman rank correlation coefficients
    and p-values for each subnetwork-environment pair
- name: pandas
  role: Loads and joins the subnetwork richness table with the environmental time-series
    and assembles the correlation results table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:badceb02e035b7d556ca2a2e63da576a6f744bf32688e17c88035606564544fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - WGCNA
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# subnetwork-environment-correlation-analysis

## Summary

Computes Spearman rank correlations between WGCNA subnetwork taxon-richness/member profiles and measured environmental time-series (e.g., chlorophyll a, sea surface temperature) from the Monterey Bay eDNA study, to test whether seasonally clustered communities track environmental properties over time.

## When to use

Use when a community network analysis has produced subnetwork (module) memberships with richness or abundance profiles and you need to test their association with measured environmental variables such as chlorophyll a or sea surface temperature.

## When NOT to use

- Input is raw eDNA sequence data or an unclustered abundance feature table — subnetwork memberships must already exist from the upstream WGCNA community-network-analysis step
- You need to infer causal or direct predator-prey interactions from co-occurrence — the article notes it is unclear whether co-occurrence reflects direct interactions or a common response to an external driver

## Inputs

- WGCNA subnetwork membership table with per-module taxon richness/member profiles (chained from upstream community-network-analysis)
- Monterey Bay environmental variable time-series (sea surface temperature, chlorophyll a, and other key environmental/biotic variables)

## Outputs

- Results table of Spearman correlation coefficients and p-values for every subnetwork-environment pair
- Validated correlation statistics for grey-subnetwork vs chlorophyll a (r = -0.76, p = 0.03) and blue-subnetwork vs sea surface temperature (r = 0.74, p = 0.04)

## How to apply

Load the WGCNA subnetwork membership and richness/member-profile table (chained from the upstream community-network-analysis step of the 18-month Monterey Bay eDNA time-series) alongside the environmental variable time-series (sea surface temperature, chlorophyll a). Compute Spearman rank correlations between each subnetwork richness/member profile and each environmental variable using scipy.stats.spearmanr (or R cor.test with method='spearman'), since the relationships need not be linear. Record the correlation coefficient and p-value for every subnetwork-environment pair in a results table. Extract and report the key pairs — grey (winter) subnetwork vs chlorophyll a and blue (autumn) subnetwork vs sea surface temperature — comparing recomputed values to the published statistics to validate the pipeline.

## Related tools

- **WGCNA** (Upstream producer of the subnetwork memberships and richness/member-profile tables that are correlated against environmental variables)
- **scipy** (scipy.stats.spearmanr computes the Spearman rank correlation coefficients and p-values for each subnetwork-environment pair)
- **pandas** (Loads and joins the subnetwork richness table with the environmental time-series and assembles the correlation results table)

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(richness['grey'], env['chlorophyll_a'])  # expect r=-0.76, p=0.03
```

## Evaluation signals

- Recomputed grey-subnetwork vs chlorophyll a Spearman r = -0.76 with p = 0.03
- Recomputed blue-subnetwork vs sea surface temperature Spearman r = 0.74 with p = 0.04
- Results table contains a coefficient and p-value for every subnetwork-environment combination tested
- Correlation direction matches the published finding: winter (grey) richness associated with LOWER chlorophyll a; autumn (blue) richness associated with HIGHER sea surface temperature

## Limitations

- Whether taxon co-occurrence is driven by direct predator-prey relationships or a common response to an external driver remains unclear
- Network connectivity does not necessarily indicate interaction; longer time-series are needed to increase power and validity of the network-based approach
- No method, including this one, identifies all taxa in an ecosystem, and amplification is subject to primer biases and inhibitors, affecting which taxa are detected

## Evidence

- [results] grey vs chlorophyll a and blue vs SST correlations: [evidence span withheld — anchor status: not_found]
- [results] blue subnetwork and SST: [evidence span withheld — anchor status: not_found]
- [intro] objective linking communities to environmental variables: "test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a"
- [discussion] subnetworks fluctuate over time: "groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
