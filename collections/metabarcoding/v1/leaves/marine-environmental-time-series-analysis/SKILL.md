---
name: marine-environmental-time-series-analysis
description: Use when a chained community-network-analysis step has produced subnetwork
  membership/richness tables and you must test each subnetwork's association with
  measured environmental time-series such as SST or chlorophyll a.
when_to_use_negative:
- Input is raw sequencing data or an unclustered feature table — the subnetwork membership
  must already exist from the upstream WGCNA community-network-analysis step
- Environmental variables and eDNA time points are not sample-aligned (no matched
  dates), since rank correlation requires paired observations
- The goal is inferring direct predator-prey interactions from co-occurrence rather
  than environment-richness associations — the article notes co-occurrence may reflect
  'a common response to an external driver'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: scipy
  role: Provides scipy.stats.spearmanr to compute Spearman rank correlation coefficients
    and p-values between subnetwork richness and environmental variables
  repo: github.com/marinebon/eDNA_microbes_whales
- name: pandas
  role: Loads and aligns the subnetwork richness table with the environmental variable
    time-series and assembles the per-pair results table
  repo: github.com/marinebon/eDNA_microbes_whales
- name: WGCNA
  role: Upstream producer of the subnetwork membership and richness/member-profile
    tables that are correlated against environmental variables
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:902a98d42b88f34704fef9cb735c7bf8e6ca068607b6068039259059e6b77021
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - WGCNA
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# marine-environmental-time-series-analysis

## Summary

Statistical correlation of WGCNA subnetwork taxon richness/member profiles from an eDNA seawater time-series against measured Monterey Bay environmental variables (chlorophyll a, sea surface temperature) using Spearman rank correlation, to test whether seasonal subnetworks track environmental drivers.

## When to use

Use when a chained community-network-analysis step has produced subnetwork membership/richness tables and you must test each subnetwork's association with measured environmental time-series such as SST or chlorophyll a.

## When NOT to use

- Input is raw sequencing data or an unclustered feature table — the subnetwork membership must already exist from the upstream WGCNA community-network-analysis step
- Environmental variables and eDNA time points are not sample-aligned (no matched dates), since rank correlation requires paired observations
- The goal is inferring direct predator-prey interactions from co-occurrence rather than environment-richness associations — the article notes co-occurrence may reflect 'a common response to an external driver'

## Inputs

- WGCNA subnetwork membership table mapping taxa to seasonal subnetworks (e.g. grey/winter, blue/autumn)
- Subnetwork taxon richness or member-abundance profile per time point
- Monterey Bay environmental time-series: sea surface temperature and chlorophyll a, aligned to eDNA sampling dates (2015-2016)

## Outputs

- Results table of Spearman correlation coefficient (r) and p-value for every subnetwork-environment pair
- Validated rows for grey-subnetwork vs chlorophyll a and blue-subnetwork vs sea surface temperature matching reported statistics

## How to apply

Load the WGCNA subnetwork membership and richness/member-profile table (chained from the upstream community-network-analysis sub-task) together with the Monterey Bay environmental variable time-series (sea surface temperature, chlorophyll a). Compute Spearman rank correlations between each subnetwork richness/member profile and each measured environmental variable using scipy.stats.spearmanr (or R cor.test with method='spearman'), which is appropriate because the variables are non-parametric ranks over the 18-month (2015-2016) sampling period. Record the correlation coefficient (r) and p-value for every subnetwork-environment pair in a results table so all pairwise tests are auditable. Extract the grey-subnetwork vs chlorophyll a and blue-subnetwork vs sea surface temperature rows and compare their r and p values to the reported statistics (grey vs chlorophyll a r = -0.76, p = 0.03; blue vs SST r = 0.74, p = 0.04). Rationale: the paper tested 'the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a, and other key environmental and biotic variables', and found the winter (grey) subnetwork tracks lower chlorophyll a while the autumn (blue) subnetwork tracks higher sea surface temperatures.

## Related tools

- **scipy** (Provides scipy.stats.spearmanr to compute Spearman rank correlation coefficients and p-values between subnetwork richness and environmental variables) — github.com/marinebon/eDNA_microbes_whales
- **pandas** (Loads and aligns the subnetwork richness table with the environmental variable time-series and assembles the per-pair results table) — github.com/marinebon/eDNA_microbes_whales
- **WGCNA** (Upstream producer of the subnetwork membership and richness/member-profile tables that are correlated against environmental variables)

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(richness['grey_subnetwork'], env['chlorophyll_a'])  # expect r=-0.76, p=0.03
```

## Evaluation signals

- Recomputed grey-subnetwork vs chlorophyll a Spearman r = -0.76 with p = 0.03 matches the reported statistic
- Recomputed blue-subnetwork vs sea surface temperature Spearman r = 0.74 with p = 0.04 matches the reported statistic
- Results table contains exactly one r/p row for every subnetwork-environment combination (completeness check)
- Correlation inputs have equal-length, sample-aligned vectors (no NaNs or date mismatches between eDNA time points and environmental measurements)
- Statistical method is rank-based (Spearman), not Pearson, as specified in the validation criteria

## Limitations

- Network connectivity does not necessarily indicate interaction — correlations between subnetworks and environmental variables do not prove direct mechanistic links
- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Amplification bias, inhibitors, and variable template abundances affect taxon detection, so richness values fed into the correlations are method-dependent
- No method, including this one, identifies all taxa in an ecosystem, so richness-based correlations may miss undetected taxa
- Longer time-series are needed to increase the power and validity of eDNA-based biodiversity trend analyses

## Evidence

- [results] grey vs chlorophyll a correlation: [evidence span withheld — anchor status: not_found]
- [results] blue vs SST correlation: [evidence span withheld — anchor status: not_found]
- [intro] objective statement: "This approach allowed us to test hypotheses about the predicted association between the richness of taxa and environmental variables, including sea surface temperature, chlorophyll a"
- [discussion] caveat on co-occurrence: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
