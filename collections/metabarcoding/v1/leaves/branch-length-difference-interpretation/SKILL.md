---
name: branch-length-difference-interpretation
description: Use when you have HMP-style V1V3 or V3V5 16S OTUs with precomputed MSD
  and BLD dissimilarities between each representative sequence and its OTU members,
  and you need to test whether BLD reveals evolutionary divergence that MSD underestimates.
when_to_use_negative:
- You have no phylogenetic tree or branch lengths — BLD cannot be computed and only
  percent/MSD dissimilarity is available, so the MSD-vs-BLD comparison is impossible.
- Your goal is OTU annotation or classification rather than assessing whether the
  97% PSD-based clustering is evolutionarily compact.
- The OTUs were not built from percent sequence dissimilarity (PSD) at a ~97% threshold,
  so the MSD/BLD-vs-0.03-threshold interpretation does not apply.
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME
  role: Pipeline used to generate the HMP OTUs (97% sequence similarity threshold)
    whose representative-to-member dissimilarities are analyzed
- name: phylo
  role: Companion repository for the paper supplying phylogenetic distance (BLD) computation
    and analysis scripts
  repo: github.com/namphuon/phylo
- name: scipy.stats.pearsonr
  role: Computes the read-level and OTU-level Pearson correlation coefficients between
    MSD and BLD dissimilarity vectors
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:95ef83aa28967f4aadc722760ffbf18535f9ca494c46b78b5365a2e7a453194a
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
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# branch-length-difference-interpretation

## Summary

A method for quantifying how phylogenetic branch length distance (BLD) complements and corrects multiple sequence alignment-based sequence dissimilarity (MSD) when assessing 16S rRNA OTU compactness in HMP V1V3/V3V5 amplicon data, by correlating read-level and per-OTU-mean MSD/BLD pairs.

## When to use

Use when you have HMP-style V1V3 or V3V5 16S OTUs with precomputed MSD and BLD dissimilarities between each representative sequence and its OTU members, and you need to test whether BLD reveals evolutionary divergence that MSD underestimates.

## When NOT to use

- You have no phylogenetic tree or branch lengths — BLD cannot be computed and only percent/MSD dissimilarity is available, so the MSD-vs-BLD comparison is impossible.
- Your goal is OTU annotation or classification rather than assessing whether the 97% PSD-based clustering is evolutionarily compact.
- The OTUs were not built from percent sequence dissimilarity (PSD) at a ~97% threshold, so the MSD/BLD-vs-0.03-threshold interpretation does not apply.

## Inputs

- HMP V1V3 OTU dataset (representative sequences plus member reads)
- HMP V3V5 OTU dataset (representative sequences plus member reads)
- Precomputed pairwise MSD dissimilarity values between each representative sequence and all member sequences
- Precomputed pairwise BLD dissimilarity values between each representative sequence and all member sequences

## Outputs

- correlation_results.csv with four Pearson correlation coefficients plus region (V1V3/V3V5) and level (read/OTU) labels
- Interpretation of whether OTU-level correlation exceeds read-level correlation, indicating BLD captures evolutionary diversity that MSD underestimates

## How to apply

Load the HMP V1V3 and V3V5 OTU datasets together with the precomputed pairwise MSD and BLD dissimilarity values between each OTU representative sequence and all of its member sequences. For each hypervariable region separately, assemble paired (MSD, BLD) dissimilarity vectors across all read-to-representative pairs and compute the read-level Pearson correlation with scipy.stats.pearsonr. Then aggregate dissimilarities per OTU (mean MSD and mean BLD per OTU) and recompute the Pearson correlation at the OTU level. Interpret OTU-level correlation exceeding read-level correlation (V3V5: 0.41 vs 0.10; V1V3: 0.63 vs 0.38) as evidence that averaging suppresses noise and that BLD uncovers evolutionary divergence masked by MSD. Write the four coefficients plus region and level labels to correlation_results.csv.

## Related tools

- **QIIME** (Pipeline used to generate the HMP OTUs (97% sequence similarity threshold) whose representative-to-member dissimilarities are analyzed)
- **phylo** (Companion repository for the paper supplying phylogenetic distance (BLD) computation and analysis scripts) — github.com/namphuon/phylo
- **scipy.stats.pearsonr** (Computes the read-level and OTU-level Pearson correlation coefficients between MSD and BLD dissimilarity vectors)

## Examples

```
from scipy.stats import pearsonr; r_v3v5_otu, _ = pearsonr(mean_msd_per_otu, mean_bld_per_otu)
```

## Evaluation signals

- Read-level Pearson r matches the reported structure: ~0.10 for V3V5 and ~0.38 for V1V3.
- OTU-level (per-OTU mean MSD vs mean BLD) Pearson r matches ~0.41 for V3V5 and ~0.63 for V1V3.
- OTU-level correlation exceeds read-level correlation for both regions, confirming the paper's pattern that averaging reveals BLD-MSD agreement.
- correlation_results.csv contains exactly four coefficients, each labeled with region (V1V3 or V3V5) and level (read or OTU).
- All MSD/BLD pairs are restricted to representative-to-member pairs within OTUs (no cross-OTU comparisons), consistent with the paper's within-OTU design.

## Limitations

- MSD can underestimate evolutionary divergence: percent sequence similarity can overestimate evolutionary similarity, so BLD-based reads of divergence may exceed MSD-based ones even within a single OTU.
- OTU compactness varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold within a PSD-defined OTU, so conclusions do not transfer uniformly across taxa.
- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation, capping the interpretability of any PSD-based OTU structure.

## Evidence

- [intro] Objective: compare three dissimilarity metrics (PSD, MSD, BLD) for quantifying evolutionary distance between pairs of sequences.: "We will discuss three different dissimilarity metrics for quantifying the evolutionary distance between pairs of sequences"
- [intro] Within-OTU MSD/BLD computation design: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] Compactness threshold context: "We will show that the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU."
- [results] Empirical scale of MSD exceedance in V3V5 OTUs: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
