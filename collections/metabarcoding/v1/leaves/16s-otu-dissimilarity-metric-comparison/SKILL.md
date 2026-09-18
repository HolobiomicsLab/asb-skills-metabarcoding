---
name: 16s-otu-dissimilarity-metric-comparison
description: Use when 16S rRNA amplicon reads have already been clustered into OTUs
  (e.g. QIIME, 97% PSD threshold) and you need to re-evaluate OTU compactness or quantify
  how strongly MSD and BLD dissimilarities correlate, at read level or aggregated
  per OTU.
when_to_use_negative:
- Input is raw unclustered amplicon reads — OTUs must already exist (the article uses
  QIIME-based OTUs from the HMP computed with PSD at 97% sequence similarity)
- You need de novo phylogenetic placement or tree building — this skill only compares
  precomputed MSD/BLD dissimilarities, it does not compute phylogenies from scratch
- The analysis concerns non-16S marker genes — the article's claims (e.g. 80.5% of
  V3V5 OTUs with MSD > 0.03 from the representative) are specific to 16S rRNA hypervariable
  regions V1V3 and V3V5
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: QIIME
  role: Pipeline used to generate the HMP V1V3 and V3V5 OTU datasets at the 97% sequence
    similarity threshold that this skill re-analyzes
- name: phylo (namphuon/phylo)
  role: Repository accompanying the article for MSD/BLD dissimilarity computation
    on 16S OTU sequences
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:b5cd8c9c3075cfca7bddfeea25fe1b905b44451b500ed336dc8f87da4390641a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - phylo (namphuon/phylo)
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# 16s-otu-dissimilarity-metric-comparison

## Summary

Compares three 16S rRNA dissimilarity metrics — percent sequence similarity (PSD), MSA-based sequence dissimilarity (MSD), and phylogenetic branch length distance (BLD) — across HMP V1V3 and V3V5 OTU members to quantify how much evolutionary divergence the 97% similarity threshold and MSD hide relative to BLD.

## When to use

Use when 16S rRNA amplicon reads have already been clustered into OTUs (e.g. QIIME, 97% PSD threshold) and you need to re-evaluate OTU compactness or quantify how strongly MSD and BLD dissimilarities correlate, at read level or aggregated per OTU.

## When NOT to use

- Input is raw unclustered amplicon reads — OTUs must already exist (the article uses QIIME-based OTUs from the HMP computed with PSD at 97% sequence similarity)
- You need de novo phylogenetic placement or tree building — this skill only compares precomputed MSD/BLD dissimilarities, it does not compute phylogenies from scratch
- The analysis concerns non-16S marker genes — the article's claims (e.g. 80.5% of V3V5 OTUs with MSD > 0.03 from the representative) are specific to 16S rRNA hypervariable regions V1V3 and V3V5

## Inputs

- HMP V1V3 and V3V5 OTU datasets (QIIME-generated OTUs, 97% PSD threshold)
- Precomputed pairwise MSD and BLD dissimilarity values between each OTU representative sequence and all member sequences

## Outputs

- correlation_results.csv with four Pearson correlation coefficients plus region (V1V3/V3V5) and level (read/OTU) labels
- Per-region read-level and OTU-level MSD-BLD correlation summary (r = 0.10 and 0.41 for V3V5; r = 0.38 and 0.63 for V1V3)

## How to apply

Take the HMP V1V3 and V3V5 OTU datasets and precomputed pairwise MSD and BLD dissimilarity values between each representative sequence and all member sequences. Assemble paired (MSD, BLD) dissimilarity vectors across all read-to-representative pairs per hypervariable region and compute read-level Pearson correlations with scipy.stats.pearsonr. Then aggregate to the OTU level (per-OTU mean MSD and mean BLD) and recompute the Pearson correlation per region; expect OTU-level coefficients of 0.41 (V3V5) and 0.63 (V1V3) versus weak read-level correlations (r = 0.10 V3V5, 0.38 V1V3). The rationale is that 97% PSD is only a rough approximation of species boundaries and can overestimate evolutionary similarity, so BLD is used to reveal evolutionary divergence that MSD underestimates.

## Related tools

- **QIIME** (Pipeline used to generate the HMP V1V3 and V3V5 OTU datasets at the 97% sequence similarity threshold that this skill re-analyzes)
- **phylo (namphuon/phylo)** (Repository accompanying the article for MSD/BLD dissimilarity computation on 16S OTU sequences) — github.com/namphuon/phylo

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(msd, bld)  # read-level, then aggregate per-OTU means and repeat
```

## Evaluation signals

- Read-level Pearson r between MSD and BLD reproduces 0.10 for V3V5 and 0.38 for V1V3
- OTU-level (per-OTU mean MSD and mean BLD) Pearson r reproduces 0.41 for V3V5 and 0.63 for V1V3, i.e. correlation increases upon aggregation
- Dissimilarity pair counts match the number of read-to-representative pairs in each region's OTU datasets
- correlation_results.csv contains exactly four rows (region × level) with valid r values in [-1, 1]
- Cross-check that a substantial fraction of OTU sequences exceed the 0.03 dissimilarity threshold from their representative (e.g. 80.5% of V3V5 OTUs had at least one sequence with MSD > 0.03)

## Limitations

- MSD underestimates evolutionary divergence relative to BLD, so read-level MSD-BLD correlations are weak and conclusions should be drawn from the OTU-level aggregation
- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation, so OTU membership itself can bias the dissimilarity distributions
- Percent sequence similarity can overestimate evolutionary similarity between pairs of sequences, and pairwise sequence alignment-based similarity computation can result in poorly clustered OTUs whose members diverge by MSD > 0.03 from the representative

## Evidence

- [intro] We will discuss three different dissimilarity metrics for quantifying the evolutionary distance between pairs of sequences: "We will discuss three different dissimilarity metrics for quantifying the evolutionary distance between pairs of sequences"
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] Compute MSD and BLD between the representative sequence and all remaining sequences within the HMP V1V3 and V3V5 OTUs: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
