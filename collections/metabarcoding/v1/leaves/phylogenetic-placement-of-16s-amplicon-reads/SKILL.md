---
name: phylogenetic-placement-of-16s-amplicon-reads
description: Use when 16S amplicon reads from pre-clustered OTUs (e.g., HMP V1V3/V3V5
  at a 97% similarity threshold) must be placed onto a fixed reference tree to compute
  MSD and BLD between a representative sequence and all remaining OTU members.
when_to_use_negative:
- The goal is to define OTUs de novo by percent sequence similarity (PSD) clustering
  at 97% — this skill instead evaluates already-clustered OTUs with MSD and BLD.
- Sequences cannot be placed on a suitable fixed reference phylogeny (placement via
  pplacer/SEPP is a prerequisite for the MSD alignment and BLD tree).
- Only taxonomic annotation of a representative sequence is needed; placement-based
  dissimilarity measurement is unnecessary for representative-sequence annotation
  workflows.
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
tools:
- name: pplacer
  role: Maximum-likelihood and Bayesian phylogenetic placement of OTU member reads
    onto the fixed reference tree to induce the per-OTU multiple sequence alignment
  repo: github.com/namphuon/phylo
- name: SEPP
  role: SATé-enabled phylogenetic placement used with pplacer for placing short 16S
    amplicon reads onto the reference phylogeny
  repo: github.com/namphuon/phylo
- name: FastTree 2
  role: Estimates the approximate maximum-likelihood tree for each OTU alignment from
    which per-read BLD to the representative is computed
- name: PASTA
  role: Ultra-large multiple sequence alignment used in the reference alignment/phylogeny
    construction underlying placement
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:5e6f947a4cc5ab42a5f216810d634d2b19f6bbdc60106e2b308289eca82e7c31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - FastTree 2
  - PASTA
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# phylogenetic-placement-of-16s-amplicon-reads

## Summary

Places 16S rRNA amplicon reads (e.g., HMP V1V3 and V3V5 OTU members) onto a fixed reference phylogeny using pplacer/SEPP to obtain a placement-induced multiple sequence alignment per OTU, enabling maximum sequence dissimilarity (MSD) and branch length distance (BLD) computation against each OTU's representative sequence.

## When to use

Use when 16S amplicon reads from pre-clustered OTUs (e.g., HMP V1V3/V3V5 at a 97% similarity threshold) must be placed onto a fixed reference tree to compute MSD and BLD between a representative sequence and all remaining OTU members.

## When NOT to use

- The goal is to define OTUs de novo by percent sequence similarity (PSD) clustering at 97% — this skill instead evaluates already-clustered OTUs with MSD and BLD.
- Sequences cannot be placed on a suitable fixed reference phylogeny (placement via pplacer/SEPP is a prerequisite for the MSD alignment and BLD tree).
- Only taxonomic annotation of a representative sequence is needed; placement-based dissimilarity measurement is unnecessary for representative-sequence annotation workflows.

## Inputs

- HMP-derived V1V3 and V3V5 OTU member sequences (16S rRNA amplicon reads)
- Representative sequence for each OTU (single sequence selected per OTU cluster)
- Fixed reference phylogeny/alignment onto which reads are placed (for pplacer/SEPP)

## Outputs

- Placement-induced multiple sequence alignment per OTU
- Per-read MSD values (0-1 dissimilarity) between representative and each remaining OTU member
- Per-read BLD values from representative to each member read via the OTU ML tree (FastTree 2)
- Aggregated table msd_bld_dissimilarities.csv covering every OTU of both V1V3 and V3V5 regions

## How to apply

Load the OTU sequences (here, HMP-derived V1V3 and V3V5 OTUs) along with each OTU's pre-selected representative sequence, then place member reads onto a fixed reference phylogeny using pplacer (maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree) or SEPP (SATé-enabled phylogenetic placement), which together yield a placement-induced multiple sequence alignment per OTU. From each placed alignment, compute the MSD (maximum sequence dissimilarity, a 0-1 metric) between the representative sequence and every remaining read. Estimate a maximum-likelihood phylogeny per OTU alignment with FastTree 2 and read BLD (branch length dissimilarity) from the representative to each member off that tree. Aggregate per-read MSD and BLD values for every OTU of both hypervariable regions (V1-V3, V3-V5) into msd_bld_dissimilarities.csv. The rationale is that percent sequence similarity (PSD) overestimates evolutionary similarity, so MSD and BLD re-measure how compact PSD-defined OTUs really are relative to the 0.03 dissimilarity threshold.

## Related tools

- **pplacer** (Maximum-likelihood and Bayesian phylogenetic placement of OTU member reads onto the fixed reference tree to induce the per-OTU multiple sequence alignment) — github.com/namphuon/phylo
- **SEPP** (SATé-enabled phylogenetic placement used with pplacer for placing short 16S amplicon reads onto the reference phylogeny) — github.com/namphuon/phylo
- **FastTree 2** (Estimates the approximate maximum-likelihood tree for each OTU alignment from which per-read BLD to the representative is computed)
- **PASTA** (Ultra-large multiple sequence alignment used in the reference alignment/phylogeny construction underlying placement)

## Evaluation signals

- Per-read MSD values must fall within the 0-1 dissimilarity range.
- Recomputed summary aggregates from msd_bld_dissimilarities.csv must be consistent with the reported statistic that 80.5% of V3V5 OTUs contain at least one read with MSD>0.03 from the representative.
- Every read in each OTU has both an MSD and a BLD value relative to the OTU's representative sequence (no missing representative-to-member pairs).
- BLD values are derived from a per-OTU ML tree (FastTree 2) built on the placement-induced alignment, not from an unrelated tree.

## Limitations

- The traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs, so OTU compactness measured by MSD and BLD often exceeds the 0.03 threshold.
- Because OTUs are annotated based on a single representative sequence, poorly clustered OTUs can propagate mis-annotation to all member sequences.
- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation.

## Evidence

- [abstract] V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree: "V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [abstract] FastTree 2-approximately maximum-likelihood trees for large alignments: "FastTree 2-approximately maximum-likelihood trees for large alignments"
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
