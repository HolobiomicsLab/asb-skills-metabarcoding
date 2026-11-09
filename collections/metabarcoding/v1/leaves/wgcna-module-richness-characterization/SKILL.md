---
name: wgcna-module-richness-characterization
description: Use when WGCNA subnetworks (e.g. seasonal 'grey' and 'blue' modules)
  from a Monterey Bay-style eDNA co-occurrence network must be characterized by taxon
  richness and correlated against environmental variables such as chlorophyll a or
  sea surface temperature.
when_to_use_negative:
- Upstream WGCNA subnetworks have not yet been derived — this skill assumes module
  membership from a completed community network analysis
- The data are not aligned as matched time-series (module profiles and environmental
  measurements sampled over the same 2015-2016 time points)
- The goal is to validate network connectivity as direct predator-prey interaction
  — co-occurrence may instead reflect a common response to an external driver
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: WGCNA
  role: Provides the subnetwork (module) membership that this skill characterizes
    by richness and correlates with environmental variables
  repo: github.com/marinebon/eDNA_microbes_whales
- name: scipy
  role: scipy.stats.spearmanr computes the Spearman rank correlation coefficient and
    p-value for each subnetwork-environment pair
- name: pandas
  role: Loads the module membership, richness profiles, and environmental time-series
    and assembles the subnetwork-environment correlation results table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:fdb429dbe4dfe8f920818edf36938ede6efd990784d7335254fc7bbc92e74b7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - WGCNA
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# wgcna-module-richness-characterization

## Summary

Characterizes WGCNA-derived community subnetworks from an eDNA time-series by their per-sample taxon richness/member profiles and tests each module against measured environmental variables (chlorophyll a, sea surface temperature) with Spearman rank correlation.

## When to use

Use when WGCNA subnetworks (e.g. seasonal 'grey' and 'blue' modules) from a Monterey Bay-style eDNA co-occurrence network must be characterized by taxon richness and correlated against environmental variables such as chlorophyll a or sea surface temperature.

## When NOT to use

- Upstream WGCNA subnetworks have not yet been derived — this skill assumes module membership from a completed community network analysis
- The data are not aligned as matched time-series (module profiles and environmental measurements sampled over the same 2015-2016 time points)
- The goal is to validate network connectivity as direct predator-prey interaction — co-occurrence may instead reflect a common response to an external driver

## Inputs

- WGCNA subnetwork (module) membership table assigning taxa to seasonal modules (e.g. grey, blue)
- Per-sample taxon richness / module member-profile table from the 18-month Monterey Bay eDNA time-series (663 taxonomic groups at Family or higher)
- Environmental variable time-series: sea surface temperature, chlorophyll a (and other key environmental and biotic variables)

## Outputs

- Results table of Spearman r and p-value for every subnetwork-environment pair
- Highlighted rows for grey-subnetwork vs chlorophyll a (r = -0.76, p = 0.03) and blue-subnetwork vs sea surface temperature (r = 0.74, p = 0.04)

## How to apply

Load the WGCNA subnetwork membership table (chained from the upstream community-network-analysis step) together with the Monterey Bay environmental time-series (sea surface temperature, chlorophyll a). For each subnetwork, compute a per-sample taxon richness or member-profile vector and run scipy.stats.spearmanr (or R cor.test with method='spearman') against each measured environmental variable. Record the Spearman r and p-value for every subnetwork-environment pair in a results table. Interpret module-environment associations biologically: in the article, the winter (grey) subnetwork tracks lower chlorophyll a (r = −0.76, p = 0.03) and the autumn (blue) subnetwork tracks higher sea surface temperature (r = 0.74, p = 0.04), supporting seasonal subnetwork-environment coupling.

## Related tools

- **WGCNA** (Provides the subnetwork (module) membership that this skill characterizes by richness and correlates with environmental variables) — github.com/marinebon/eDNA_microbes_whales
- **scipy** (scipy.stats.spearmanr computes the Spearman rank correlation coefficient and p-value for each subnetwork-environment pair)
- **pandas** (Loads the module membership, richness profiles, and environmental time-series and assembles the subnetwork-environment correlation results table)

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(grey_richness, chl_a)
```

## Evaluation signals

- Recomputed grey-subnetwork vs chlorophyll a Spearman r equals -0.76 with p = 0.03
- Recomputed blue-subnetwork vs sea surface temperature Spearman r equals 0.74 with p = 0.04
- Results table contains one row (r, p) for every subnetwork-environment pair tested, with no missing comparisons
- Correlation is computed with Spearman (rank-based) method, not Pearson, matching the reported statistics

## Limitations

- Network connectivity does not necessarily indicate interaction; module-environment correlation may reflect a common response to an external driver rather than direct predator-prey relationships
- Amplification is subject to primer biases, inhibitors, and variable template abundances, which affect the number of taxa detected and hence module richness
- Longer time-series are needed to increase the power and validity of eDNA-based network and correlation analyses; borderline p-values (0.03, 0.04) rest on a modest number of time points

## Evidence

- [intro] This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a: "This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a"
- [discussion] Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time.: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
