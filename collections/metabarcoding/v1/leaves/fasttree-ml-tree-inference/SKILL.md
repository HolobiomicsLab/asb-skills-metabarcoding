---
name: fasttree-ml-tree-inference
description: Use when per-OTU nucleotide alignments exist and a fast ML tree is needed
  to quantify branch length dissimilarity (BLD) between a representative sequence
  and OTU member reads.
when_to_use_negative:
- When the analysis goal is percent sequence dissimilarity (PSD) clustering — this
  skill addresses BLD on an ML tree, not 97% sequence similarity OTU construction
- When only pairwise sequence similarity is needed and no phylogenetic tree or branch
  length information is required
- When alignments are so small that exact ML inference with RAxML is feasible and
  preferable to FastTree 2's approximate-ML approach
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: FastTree 2
  role: Estimates the approximately maximum-likelihood phylogeny for each OTU alignment
    from which BLD is computed
- name: pplacer
  role: Places member reads onto a fixed reference tree to obtain the placement-induced
    multiple sequence alignment used as tree-inference input
- name: SEPP
  role: SATé-enabled phylogenetic placement supporting the per-OTU alignment construction
- name: PASTA
  role: Ultra-large multiple sequence alignment support for building OTU alignments
- name: phylo (namphuon/phylo)
  role: Repository associated with the MSD/BLD dissimilarity computation on the estimated
    trees
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:f22f28d22a98ceddc7e669ffd7181bb7e0083d196ce53170b544c5a0457e0c72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastTree
  - FastTree 2
  - PASTA
  - SEPP
  - phylo (namphuon/phylo)
  - pplacer
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# FastTree 2 maximum-likelihood phylogeny estimation for OTU alignments

## Summary

Estimates an approximate maximum-likelihood phylogenetic tree from each OTU's placement-induced multiple sequence alignment with FastTree 2, so that branch length distance (BLD) can be computed between an OTU's representative sequence and every member read.

## When to use

Use when per-OTU nucleotide alignments exist and a fast ML tree is needed to quantify branch length dissimilarity (BLD) between a representative sequence and OTU member reads.

## When NOT to use

- When the analysis goal is percent sequence dissimilarity (PSD) clustering — this skill addresses BLD on an ML tree, not 97% sequence similarity OTU construction
- When only pairwise sequence similarity is needed and no phylogenetic tree or branch length information is required
- When alignments are so small that exact ML inference with RAxML is feasible and preferable to FastTree 2's approximate-ML approach

## Inputs

- Placement-induced multiple sequence alignment per OTU (produced by pplacer/SEPP from HMP V1V3 and V3V5 OTU sequences)
- Each OTU's designated representative sequence
- FASTA-format OTU member sequence alignments

## Outputs

- Approximately maximum-likelihood phylogenetic tree per OTU (Newick)
- Per-read BLD (branch length dissimilarity) values from representative sequence to each member read
- Aggregated per-read MSD and BLD table: msd_bld_dissimilarities.csv

## How to apply

First build the per-OTU alignment by placing HMP-derived V1V3 and V3V5 member reads onto a fixed reference phylogeny with pplacer/SEPP to obtain the placement-induced multiple sequence alignment. Then estimate a maximum-likelihood phylogeny for each OTU alignment with FastTree 2. From the resulting tree, compute the BLD between the representative sequence and each member read as the branch length dissimilarity on the tree path. Aggregate the per-read BLD values (together with per-read MSD values from the placed alignment) for every OTU of both regions and write them to msd_bld_dissimilarities.csv. FastTree 2 is chosen because it handles large alignments while producing approximately maximum-likelihood trees.

## Related tools

- **FastTree 2** (Estimates the approximately maximum-likelihood phylogeny for each OTU alignment from which BLD is computed)
- **pplacer** (Places member reads onto a fixed reference tree to obtain the placement-induced multiple sequence alignment used as tree-inference input)
- **SEPP** (SATé-enabled phylogenetic placement supporting the per-OTU alignment construction)
- **PASTA** (Ultra-large multiple sequence alignment support for building OTU alignments)
- **phylo (namphuon/phylo)** (Repository associated with the MSD/BLD dissimilarity computation on the estimated trees) — github.com/namphuon/phylo

## Examples

```
FastTree -nt otu_alignment.fasta > otu_tree.nwk
```

## Evaluation signals

- Recompute the summary aggregates from msd_bld_dissimilarities.csv and check consistency with the reported statistic that 80.5% of V3V5 OTUs contain at least one read with MSD>0.03 from the representative
- Verify per-read MSD values fall in the 0–1 dissimilarity range
- Confirm a phylogenetic tree was produced for every OTU alignment before BLD computation
- Check that BLD values are computed from the representative sequence to each member read, not between arbitrary pairs
- Confirm both V1V3 and V3V5 regions are covered in the aggregated output table

## Limitations

- FastTree 2 produces approximately maximum-likelihood trees, not exact ML trees (e.g., unlike RAxML version 8)
- BLD depends on the quality of the placement-induced alignment; pairwise alignment-based similarity computation can result in poorly clustered OTUs
- The 97% 16S rRNA sequence similarity threshold used to build the OTUs is only a rough approximation for delineating species, and both MSD and BLD often exceed the 0.03 threshold within an OTU

## Evidence

- [abstract] FastTree 2-approximately maximum-likelihood trees for large alignments: "FastTree 2-approximately maximum-likelihood trees for large alignments"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [other] Estimate a maximum-likelihood phylogeny for each OTU alignment with FastTree 2 and compute BLD (branch length dissimilarity) from the representative to each member read.: [evidence span withheld — anchor status: not_found]
