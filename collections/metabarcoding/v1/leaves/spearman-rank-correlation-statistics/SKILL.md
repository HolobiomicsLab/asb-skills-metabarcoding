---
name: spearman-rank-correlation-statistics
description: Use when a taxa-by-time or richness-by-time table has been clustered
  into subnetworks (e.g. WGCNA grey/blue modules) and you need to test each subnetwork's
  association against measured environmental variables such as chlorophyll a and SST.
when_to_use_negative:
- Input is a raw sequence/amplicon dataset that has not yet been clustered into community
  subnetworks — run the upstream WGCNA community-network-analysis first
- The relationship under test is expected to be linear and residuals normally distributed,
  warranting Pearson rather than rank-based correlation
- Environmental variables are missing or sampled on a non-matching time grid relative
  to the eDNA time-series samples
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scipy
  role: Provides scipy.stats.spearmanr used to compute Spearman rank correlations
    between subnetwork richness and environmental variables
- name: pandas
  role: Loads and joins the subnetwork richness/member-profile table with the environmental
    time-series and assembles the results table
- name: WGCNA
  role: Upstream producer of the subnetwork (grey/blue module) membership that this
    skill correlates with environmental variables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:c9bcc5e433d84ea97dfd7f219b7ea26d9dcacc9962441a1b23215f7b03ef52f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - WGCNA
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# spearman-rank-correlation-statistics

## Summary

Computes Spearman rank correlations between WGCNA subnetwork taxon richness/member profiles and Monterey Bay environmental time-series (chlorophyll a, sea surface temperature), yielding coefficient and p-value per subnetwork-environment pair.

## When to use

Use when a taxa-by-time or richness-by-time table has been clustered into subnetworks (e.g. WGCNA grey/blue modules) and you need to test each subnetwork's association against measured environmental variables such as chlorophyll a and SST.

## When NOT to use

- Input is a raw sequence/amplicon dataset that has not yet been clustered into community subnetworks — run the upstream WGCNA community-network-analysis first
- The relationship under test is expected to be linear and residuals normally distributed, warranting Pearson rather than rank-based correlation
- Environmental variables are missing or sampled on a non-matching time grid relative to the eDNA time-series samples

## Inputs

- WGCNA subnetwork membership and richness/member-profile table (from upstream community-network-analysis task)
- Monterey Bay environmental variable time-series (sea surface temperature, chlorophyll a)

## Outputs

- Results table of Spearman correlation coefficient and p-value for every subnetwork-environment pair
- Highlighted grey-subnetwork vs chlorophyll a (r = -0.76, p = 0.03) and blue-subnetwork vs SST (r = 0.74, p = 0.04) statistics

## How to apply

Load the WGCNA subnetwork membership and richness/member-profile table chained from the upstream community-network-analysis task together with the Monterey Bay environmental time-series (sea surface temperature, chlorophyll a). Compute Spearman rank correlations between each subnetwork richness/member profile and each measured environmental variable using scipy.stats.spearmanr (or R cor.test with method='spearman'), which does not assume linearity and is appropriate for the 18-month 2015-2016 time-series. Record the correlation coefficient and p-value for every subnetwork-environment pair in a results table. Extract the grey-subnetwork vs chlorophyll a and blue-subnetwork vs sea surface temperature rows and compare their r and p values against the reported statistics (r = -0.76, p = 0.03; r = 0.74, p = 0.04).

## Related tools

- **scipy** (Provides scipy.stats.spearmanr used to compute Spearman rank correlations between subnetwork richness and environmental variables)
- **pandas** (Loads and joins the subnetwork richness/member-profile table with the environmental time-series and assembles the results table)
- **WGCNA** (Upstream producer of the subnetwork (grey/blue module) membership that this skill correlates with environmental variables)

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(subnetwork_richness['grey'], env['chlorophyll_a'])
```

## Evaluation signals

- Recomputed grey-subnetwork vs chlorophyll a Spearman r equals -0.76 with p = 0.03
- Recomputed blue-subnetwork vs sea surface temperature Spearman r equals 0.74 with p = 0.04
- Results table contains a coefficient and p-value for every subnetwork-environment pair tested
- Correlations are computed on matched time points of the 18-month (2015-2016) Monterey Bay series (no length mismatch between richness and environmental vectors)

## Limitations

- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear — correlation does not establish mechanism
- Network connectivity does not necessarily indicate interaction, so subnetwork-environment correlations should be treated as hypothesis-generating and followed by hypothesis-based research and experiments
- Longer time-series are needed to increase the power and validity of the technique

## Evidence

- [results] finding quote: [evidence span withheld — anchor status: not_found]
- [intro] objective quote: "This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a"
- [readme] readme quote: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [discussion] discussion caveat: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
