---
name: msd-computation-from-placement-msa
description: Use when auditing compactness of 16S rRNA OTUs (e.g. HMP V1V3/V3V5 OTUs
  clustered at 97% PSD) by re-measuring each member's distance from the representative
  with alignment-based (MSD) and tree-based (BLD) metrics.
when_to_use_negative:
- The goal is to define OTUs de novo by percent sequence similarity (PSD) clustering
  at 97% — this skill audits existing OTUs rather than building them
- Input sequences lack OTU grouping or a designated representative sequence, so no
  representative-vs-member comparison can be defined
- A different dissimilarity metric (e.g. PSD) is explicitly requested, which this
  skill does not compute
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of OTU member reads onto a fixed
    reference tree to induce the per-OTU MSA used for MSD
- name: SEPP
  role: SATé-enabled phylogenetic placement for inserting the 16S amplicon reads into
    the reference alignment/tree
- name: FastTree 2
  role: Estimates the approximate maximum-likelihood phylogeny on each per-OTU alignment
    from which BLD to the representative is read off
- name: PASTA
  role: Ultra-large multiple sequence alignment used to build the OTU alignments underlying
    MSD and tree estimation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:292c79242d80cf4396e3b85783c47ad692736af8a8938dc9ad6067da4f5f9607
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastTree 2
  - PASTA
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# MSD computation from placement MSA

## Summary

Computes maximum sequence dissimilarity (MSD) — and phylogenetic branch length distance (BLD) — between each HMP OTU's representative sequence and every remaining member read, using a placement-induced multiple sequence alignment and an ML tree per OTU. It quantifies how far OTU members diverge from the single annotated representative.

## When to use

Use when auditing compactness of 16S rRNA OTUs (e.g. HMP V1V3/V3V5 OTUs clustered at 97% PSD) by re-measuring each member's distance from the representative with alignment-based (MSD) and tree-based (BLD) metrics.

## When NOT to use

- The goal is to define OTUs de novo by percent sequence similarity (PSD) clustering at 97% — this skill audits existing OTUs rather than building them
- Input sequences lack OTU grouping or a designated representative sequence, so no representative-vs-member comparison can be defined
- A different dissimilarity metric (e.g. PSD) is explicitly requested, which this skill does not compute

## Inputs

- HMP-derived V1V3 OTU member sequences (FASTA)
- HMP-derived V3V5 OTU member sequences (FASTA)
- Representative sequence for each OTU
- Fixed reference phylogeny for phylogenetic placement (pplacer/SEPP)
- Per-OTU placement-induced multiple sequence alignments

## Outputs

- Per-read MSD and BLD dissimilarity table (msd_bld_dissimilarities.csv) covering every OTU of both regions
- Summary statistics, e.g. fraction of OTUs containing at least one read with MSD > 0.03 from the representative

## How to apply

Load the HMP-derived V1V3 and V3V5 OTU sequences together with each OTU's representative sequence. Place member reads onto a fixed reference phylogeny with pplacer/SEPP to obtain a placement-induced multiple sequence alignment per OTU (PASTA was used for large-scale alignment support in the paper). From each placed alignment, compute MSD between the representative and every remaining read as a dissimilarity in the 0–1 range, and separately estimate a maximum-likelihood phylogeny per OTU alignment with FastTree 2 to derive BLD from the representative to each member. Aggregate all per-read MSD and BLD values across OTUs of both regions into msd_bld_dissimilarities.csv. Rationale: percent sequence similarity (PSD) can overestimate evolutionary similarity, so re-scoring OTUs with MSD/BLD against the 0.03 dissimilarity threshold reveals non-compact OTUs that would otherwise inherit one annotation from the representative.

## Related tools

- **pplacer** (Maximum-likelihood phylogenetic placement of OTU member reads onto a fixed reference tree to induce the per-OTU MSA used for MSD)
- **SEPP** (SATé-enabled phylogenetic placement for inserting the 16S amplicon reads into the reference alignment/tree)
- **FastTree 2** (Estimates the approximate maximum-likelihood phylogeny on each per-OTU alignment from which BLD to the representative is read off)
- **PASTA** (Ultra-large multiple sequence alignment used to build the OTU alignments underlying MSD and tree estimation)

## Examples

```
FastTree -nt otu_alignment.fasta > otu_ml.tre
```

## Evaluation signals

- All per-read MSD values fall in the 0–1 dissimilarity range
- Recomputing summary aggregates from msd_bld_dissimilarities.csv reproduces the reported statistic that 80.5% of V3V5 OTUs contain at least one read with MSD > 0.03 from the representative
- Aggregate check matches the reported 2.7% of all V3V5 sequences having MSD > 3% from the representative
- BLD values are non-negative and correspond to paths in the FastTree 2 ML tree estimated from the same OTU alignment
- Every OTU member read appears exactly once per OTU in the output table alongside its representative

## Limitations

- MSD depends on the placement-induced MSA quality; the paper notes that traditional pairwise alignment-based similarity computation can result in poorly clustered OTUs
- Results are sensitive to the OTU partitioning itself: the 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation
- Interpretation is per-OTU: MSD/BLD summarize divergence from a single representative sequence, so poorly clustered OTUs propagate annotation errors to all members

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] the percent sequence similarity can overestimate the evolutionary similarity between pairs of sequences: "the percent sequence similarity can overestimate the evolutionary similarity between pairs of sequences"
- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
