---
name: pearson-correlation-statistics
description: Use when you have paired pairwise dissimilarity values (e.g., MSA-based
  MSD and phylogenetic branch length BLD) between OTU representative sequences and
  their member reads, and need to quantify agreement at read vs. OTU level.
when_to_use_negative:
- Input is a raw feature table or abundance matrix rather than paired pairwise dissimilarity
  values between representative sequences and OTU members.
- The dissimilarity values have not yet been computed — run the upstream MSD/BLD distance-computation
  step (QIIME-based OTU picking at 97% followed by MSD and BLD calculation) first.
- The analysis goal is OTU clustering quality itself (e.g., measuring MSD/BLD compactness
  against the 0.03 threshold) rather than correlation between two metrics.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3299
tools:
- name: QIIME
  role: Pipeline used to generate the HMP V1V3 and V3V5 OTU datasets (OTUs computed
    using PSD at a 97% sequence similarity threshold) whose read-to-representative
    dissimilarities are correlated.
- name: scipy.stats.pearsonr
  role: Computes the read-level and OTU-level Pearson correlation coefficients between
    paired MSD and BLD dissimilarity vectors.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:ca1a6be2bdf0c39c8d60eb2509bd11e2487818ed9fd7eb950dc0a9fdd4d08f44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME
  - scipy.stats.pearsonr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pearson-correlation-statistics

## Summary

Quantifies how strongly two sequence-dissimilarity metrics (MSD-based and BLD-based) agree by computing Pearson correlation coefficients at both the individual read level and the aggregated OTU level for HMP 16S hypervariable regions (V1V3 and V3V5), revealing evolutionary divergence that percent sequence similarity underestimates.

## When to use

Use when you have paired pairwise dissimilarity values (e.g., MSA-based MSD and phylogenetic branch length BLD) between OTU representative sequences and their member reads, and need to quantify agreement at read vs. OTU level.

## When NOT to use

- Input is a raw feature table or abundance matrix rather than paired pairwise dissimilarity values between representative sequences and OTU members.
- The dissimilarity values have not yet been computed — run the upstream MSD/BLD distance-computation step (QIIME-based OTU picking at 97% followed by MSD and BLD calculation) first.
- The analysis goal is OTU clustering quality itself (e.g., measuring MSD/BLD compactness against the 0.03 threshold) rather than correlation between two metrics.

## Inputs

- HMP V1V3 and V3V5 OTU datasets (reads clustered at 97% sequence similarity with representative sequences)
- Precomputed pairwise MSD (MSA-based sequence dissimilarity) values between each representative sequence and all member sequences
- Precomputed pairwise BLD (phylogenetic branch length distance) values between each representative sequence and all member sequences

## Outputs

- correlation_results.csv with four Pearson correlation coefficients plus region (V1V3/V3V5) and level (read/OTU) labels
- Read-level Pearson r per region (expected ~0.10 for V3V5, ~0.38 for V1V3)
- OTU-level Pearson r per region (expected ~0.41 for V3V5, ~0.63 for V1V3)

## How to apply

Load the HMP V1V3 and V3V5 OTU datasets together with precomputed pairwise MSD and BLD dissimilarity values between each representative sequence and all member sequences (computed after clustering at the 97% sequence-similarity threshold). For each hypervariable region separately, assemble paired (MSD, BLD) vectors across all read-to-representative pairs and compute the read-level Pearson correlation with scipy.stats.pearsonr. Then aggregate to the OTU level by taking per-OTU mean MSD and mean BLD and recompute the correlation. Compare against the expected structure (read-level r of 0.10 for V3V5 and 0.38 for V1V3; OTU-level r of 0.41 for V3V5 and 0.63 for V1V3) to confirm that BLD uncovers evolutionary divergence MSD underestimates. Write the four coefficients plus region and level labels to correlation_results.csv.

## Related tools

- **QIIME** (Pipeline used to generate the HMP V1V3 and V3V5 OTU datasets (OTUs computed using PSD at a 97% sequence similarity threshold) whose read-to-representative dissimilarities are correlated.)
- **scipy.stats.pearsonr** (Computes the read-level and OTU-level Pearson correlation coefficients between paired MSD and BLD dissimilarity vectors.)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(msd_vector, bld_vector)
```

## Evaluation signals

- Read-level Pearson r should be weak: approximately 0.10 for V3V5 and 0.38 for V1V3.
- OTU-level Pearson r (per-OTU mean MSD vs. mean BLD) should increase: approximately 0.41 for V3V5 and 0.63 for V1V3.
- The OTU-level r must exceed the read-level r for both regions, confirming that aggregation strengthens the MSD–BLD relationship.
- correlation_results.csv must contain exactly four rows (2 regions × 2 levels) with region and level labels plus coefficients.
- All read-to-representative pairs in each region must be represented in the paired (MSD, BLD) vectors before aggregation.

## Limitations

- Correlations are computed only on HMP V1V3 and V3V5 OTUs generated with QIIME using PSD at a 97% similarity threshold; results may not transfer to other regions or clustering methods.
- Percent sequence similarity can overestimate evolutionary similarity, so MSD–BLD correlation magnitudes reflect metrics applied to potentially poorly clustered OTUs (80.5% of V3V5 OTUs had at least one sequence with MSD>0.03 from the representative).
- OTU-level aggregation uses per-OTU means, which can mask within-OTU heterogeneity in the MSD–BLD relationship.

## Evidence

- [intro] Objective: compare three dissimilarity metrics (PSD, MSD, BLD) for quantifying evolutionary distance between pairs of sequences.: "We will discuss three different dissimilarity metrics for quantifying the evolutionary distance between pairs of sequences"
- [intro] Compute MSD and BLD between representative and member sequences in HMP V1V3 and V3V5 OTUs.: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] Within-OTU dissimilarity frequently exceeds the 0.03 threshold, motivating OTU-level correlation analysis.: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
