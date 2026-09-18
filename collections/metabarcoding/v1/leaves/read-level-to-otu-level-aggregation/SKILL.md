---
name: read-level-to-otu-level-aggregation
description: Use when you have paired read-level dissimilarity values (e.g., MSD and
  BLD from HMP V1V3/V3V5 OTUs) and weak read-level correlations, and you need to test
  whether per-OTU mean aggregation reveals stronger metric concordance.
when_to_use_negative:
- Input is already aggregated to OTU-level means with no read-to-representative dissimilarity
  pairs available to recompute the read-level baseline.
- The goal is to build or re-cluster OTUs from sequence similarity rather than to
  evaluate how dissimilarity metrics (MSD vs BLD) behave across existing OTUs.
- You need a per-sequence evolutionary divergence measure itself; this skill only
  correlates and aggregates existing MSD/BLD values.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME
  role: Pipeline used to generate the HMP V1V3 and V3V5 OTU datasets (97% sequence
    similarity clustering) whose read- and OTU-level MSD/BLD values are aggregated
    and correlated.
- name: scipy.stats.pearsonr
  role: Computes the Pearson correlation coefficients between paired MSD and BLD vectors
    at both the read level and the aggregated OTU level.
- name: phylo
  role: Repository code associated with the MSD/BLD dissimilarity computations on
    HMP OTU sequences.
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:06f7d27d0453a83b8365325067d44744eef8c8d79b97725bf896ceb13c818cc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - phylo
  - scipy.stats.pearsonr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# read-level-to-otu-level-aggregation

## Summary

Aggregates per-read dissimilarity pairs (MSD and BLD, computed between each OTU representative sequence and its member sequences) to per-OTU means and recomputes Pearson correlations, because OTU-level aggregation strengthens the observed MSD–BLD relationship and exposes evolutionary divergence that MSD underestimates.

## When to use

Use when you have paired read-level dissimilarity values (e.g., MSD and BLD from HMP V1V3/V3V5 OTUs) and weak read-level correlations, and you need to test whether per-OTU mean aggregation reveals stronger metric concordance.

## When NOT to use

- Input is already aggregated to OTU-level means with no read-to-representative dissimilarity pairs available to recompute the read-level baseline.
- The goal is to build or re-cluster OTUs from sequence similarity rather than to evaluate how dissimilarity metrics (MSD vs BLD) behave across existing OTUs.
- You need a per-sequence evolutionary divergence measure itself; this skill only correlates and aggregates existing MSD/BLD values.

## Inputs

- HMP V1V3 OTU dataset (QIIME-generated, 97% PSD threshold)
- HMP V3V5 OTU dataset (QIIME-generated, 97% PSD threshold)
- Precomputed pairwise MSD dissimilarity values between each representative sequence and all member sequences
- Precomputed pairwise BLD dissimilarity values between each representative sequence and all member sequences

## Outputs

- correlation_results.csv containing four Pearson correlation coefficients with region (V1V3, V3V5) and level (read, OTU) labels
- Per-OTU mean MSD and mean BLD values used for the aggregated correlation

## How to apply

Load the HMP V1V3 and V3V5 OTU datasets together with the precomputed pairwise MSD and BLD values between each representative sequence and all member sequences produced by the upstream distance-computation step. For each hypervariable region separately, assemble paired (MSD, BLD) vectors across all read-to-representative pairs and compute the read-level Pearson correlation with scipy.stats.pearsonr (expected roughly r=0.10 for V3V5 and r=0.38 for V1V3). Then aggregate dissimilarities to the OTU level by taking mean MSD and mean BLD per OTU and recompute the Pearson correlation per region, expecting r=0.41 for V3V5 and r=0.63 for V1V3. The rationale is that OTUs built at the 97% PSD threshold are annotated from a single representative sequence, so aggregation tests whether the OTU as a unit hides divergence that read-level noise obscures; the rise in correlation shows BLD uncovers evolutionary divergence that MSD underestimates. Write the four coefficients plus region and level labels to correlation_results.csv.

## Related tools

- **QIIME** (Pipeline used to generate the HMP V1V3 and V3V5 OTU datasets (97% sequence similarity clustering) whose read- and OTU-level MSD/BLD values are aggregated and correlated.)
- **scipy.stats.pearsonr** (Computes the Pearson correlation coefficients between paired MSD and BLD vectors at both the read level and the aggregated OTU level.)
- **phylo** (Repository code associated with the MSD/BLD dissimilarity computations on HMP OTU sequences.) — github.com/namphuon/phylo

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(mean_msd_per_otu, mean_bld_per_otu)
```

## Evaluation signals

- Read-level Pearson r should be weak: approximately 0.10 for V3V5 and 0.38 for V1V3.
- OTU-level (per-OTU mean) Pearson r should be higher: 0.41 for V3V5 and 0.63 for V1V3.
- OTU-level r must exceed the corresponding read-level r for both hypervariable regions.
- correlation_results.csv must contain exactly four rows (2 regions x 2 levels) with region and level labels.
- Each read-to-representative pair must have both a valid MSD and BLD value before aggregation; unmatched pairs indicate upstream distance-computation errors.

## Limitations

- Read-level MSD–BLD correlations are weak, so read-level comparisons alone understate the relationship between the metrics.
- OTUs were clustered at the 97% percent sequence similarity threshold, which is only a rough approximation for delineating species, so aggregation results inherit any mis-clustering.
- MSD underestimates evolutionary divergence that BLD reveals, so MSD-only summaries will miss divergence within OTUs (e.g., 80.5% of V3V5 OTUs had at least one sequence with MSD>0.03 from the representative).

## Evidence

- [results] The correlation between MSD and BLD increases when computed at the OTU level: mean MSD and BLD per OTU yield Pearson's correlation coefficients of 0.41: [evidence span withheld — anchor status: not_found]
- [intro] compute MSD and BLD between representative and all remaining sequences within HMP OTUs: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] MSD>0.03 divergence within V3V5 OTUs: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
