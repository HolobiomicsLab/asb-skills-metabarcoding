---
name: branch-length-dissimilarity-quantification
description: Use when auditing 97%-threshold 16S OTUs (e.g., HMP V1V3/V3V5) for phylogenetic
  coherence by measuring BLD between a representative sequence and all other OTU members.
when_to_use_negative:
- OTUs are already defined by phylogenetic distance rather than percent sequence similarity,
  so re-measuring BLD compactness adds no new check
- No reference phylogeny or placement framework is available and only raw pairwise
  sequence similarity (PSD) is needed
- The analysis goal is taxonomic annotation of the representative rather than assessing
  OTU compactness
edam_operation: http://edamontology.org/operation_0540
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
tools:
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of OTU member reads onto a fixed
    reference tree
  repo: github.com/namphuon/phylo
- name: SEPP
  role: SATé-enabled phylogenetic placement used to obtain the placement-induced alignment
    per OTU
  repo: github.com/namphuon/phylo
- name: FastTree 2
  role: Approximate maximum-likelihood tree estimation for each OTU alignment from
    which BLD is computed
  repo: github.com/namphuon/phylo
- name: PASTA
  role: Ultra-large multiple sequence alignment support for the OTU alignment pipeline
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:21a02800ccfe8bf7f13753c26a907a9960293aab9ecc279c31be251acdc7db92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - FastTree 2
  - PASTA
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0540
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# branch-length-dissimilarity-quantification

## Summary

Quantifies phylogenetic compactness of 16S rRNA OTUs by computing branch length distance (BLD) from each OTU's representative sequence to every member read on a maximum-likelihood tree, complementing alignment-based MSD. Used to show that OTUs clustered at 97% sequence similarity are often not compact under phylogenetic metrics.

## When to use

Use when auditing 97%-threshold 16S OTUs (e.g., HMP V1V3/V3V5) for phylogenetic coherence by measuring BLD between a representative sequence and all other OTU members.

## When NOT to use

- OTUs are already defined by phylogenetic distance rather than percent sequence similarity, so re-measuring BLD compactness adds no new check
- No reference phylogeny or placement framework is available and only raw pairwise sequence similarity (PSD) is needed
- The analysis goal is taxonomic annotation of the representative rather than assessing OTU compactness

## Inputs

- HMP-derived V1V3 and V3V5 OTU sequence sets clustered at 97% sequence similarity (PSD)
- Representative sequence selected for each OTU
- Fixed reference phylogeny for phylogenetic placement (pplacer/SEPP)

## Outputs

- Per-read BLD values from representative to each member read per OTU
- Per-read MSD values from representative to each member read per OTU
- Aggregated msd_bld_dissimilarities.csv with per-read dissimilarities for all OTUs of both regions

## How to apply

Load the HMP-derived V1V3 and V3V5 OTU sequences along with each OTU's representative sequence. Place member reads onto a fixed reference phylogeny using pplacer/SEPP to obtain a placement-induced multiple sequence alignment per OTU, then estimate a maximum-likelihood phylogeny for each OTU alignment with FastTree 2. Compute the branch length distance (BLD) between the representative sequence and each remaining member read from that ML tree, alongside the MSD (maximum sequence dissimilarity) from the placed alignment. The rationale is that percent sequence similarity (PSD) can overestimate evolutionary similarity, so re-measuring OTU compactness with BLD and MSD against the 0.03 threshold reveals OTUs whose members exceed 3% dissimilarity from the annotated representative. Aggregate per-read BLD (and MSD) values for every OTU of both regions and write them to msd_bld_dissimilarities.csv.

## Related tools

- **pplacer** (Maximum-likelihood phylogenetic placement of OTU member reads onto a fixed reference tree) — github.com/namphuon/phylo
- **SEPP** (SATé-enabled phylogenetic placement used to obtain the placement-induced alignment per OTU) — github.com/namphuon/phylo
- **FastTree 2** (Approximate maximum-likelihood tree estimation for each OTU alignment from which BLD is computed) — github.com/namphuon/phylo
- **PASTA** (Ultra-large multiple sequence alignment support for the OTU alignment pipeline) — github.com/namphuon/phylo

## Evaluation signals

- Recompute summary aggregates from msd_bld_dissimilarities.csv and confirm consistency with the reported statistic that 80.5% of V3V5 OTUs contain at least one read with MSD>0.03 from the representative
- Verify per-read MSD values fall within the 0-1 dissimilarity range
- Check that BLD is derived from an ML tree (FastTree 2) estimated on the OTU alignment, not from the pairwise alignment directly
- Confirm every OTU member read (not just a subset) has a paired BLD and MSD value relative to the representative

## Limitations

- Percent sequence similarity can overestimate evolutionary similarity between pairs of sequences, so BLD results may disagree with the 97% PSD clustering
- OTUs are annotated based on a single representative sequence, so poorly clustered OTUs propagate annotation errors to all member sequences
- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] the percent sequence similarity can overestimate the evolutionary similarity between pairs of sequences: "the percent sequence similarity can overestimate the evolutionary similarity between pairs of sequences"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [abstract] FastTree 2-approximately maximum-likelihood trees for large alignments: "FastTree 2-approximately maximum-likelihood trees for large alignments"
- [abstract] V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree: "V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree"
