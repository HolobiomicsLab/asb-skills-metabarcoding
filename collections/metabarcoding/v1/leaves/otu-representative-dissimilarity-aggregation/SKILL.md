---
name: otu-representative-dissimilarity-aggregation
description: Use when you have pre-clustered 16S rRNA OTUs (e.g. 97% PSD threshold)
  with a designated representative sequence per OTU and need per-member evolutionary
  distance metrics beyond pairwise percent similarity.
when_to_use_negative:
- Input OTUs have not yet been generated (the article clusters HMP 16S sequences at
  a 97% percent sequence similarity threshold first; this skill measures their compactness,
  it does not cluster)
- You need the three-dissimilarity-metric comparison discussion (PSD vs MSD vs BLD
  definitions) rather than concrete per-OTU MSD/BLD computation
- A representative-sequence annotation is already sufficient — this skill exists because
  annotation from a single representative can be misleading for poorly clustered OTUs
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0659
tools:
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of OTU member reads onto a fixed
    reference tree to induce the per-OTU MSA
  repo: github.com/namphuon/phylo
- name: SEPP
  role: SATé-enabled phylogenetic placement used for placing reads onto the reference
    phylogeny
  repo: github.com/namphuon/phylo
- name: FastTree 2
  role: Estimates the approximate maximum-likelihood phylogeny for each OTU alignment
    from which BLD values are computed
  repo: github.com/namphuon/phylo
- name: PASTA
  role: Ultra-large multiple sequence alignment tool used in the alignment/phylogeny
    pipeline supporting OTU alignments
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:82846bdbe5c6bf9d402e123545e533ae20ba4788986615eb85f9ca1b10496d45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - FastTree 2
  - PASTA
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# otu-representative-dissimilarity-aggregation

## Summary

Computes per-read MSD (MSA-based sequence dissimilarity) and BLD (phylogenetic branch length distance) between each HMP V1V3/V3V5 OTU's representative sequence and all remaining OTU members, quantifying how compact 97%-similarity OTUs really are.

## When to use

Use when you have pre-clustered 16S rRNA OTUs (e.g. 97% PSD threshold) with a designated representative sequence per OTU and need per-member evolutionary distance metrics beyond pairwise percent similarity.

## When NOT to use

- Input OTUs have not yet been generated (the article clusters HMP 16S sequences at a 97% percent sequence similarity threshold first; this skill measures their compactness, it does not cluster)
- You need the three-dissimilarity-metric comparison discussion (PSD vs MSD vs BLD definitions) rather than concrete per-OTU MSD/BLD computation
- A representative-sequence annotation is already sufficient — this skill exists because annotation from a single representative can be misleading for poorly clustered OTUs

## Inputs

- HMP-derived V1V3 and V3V5 OTU member sequences (16S rRNA amplicon reads)
- One representative sequence per OTU
- Reference phylogeny / OTU alignment for pplacer-SEPP placement

## Outputs

- msd_bld_dissimilarities.csv with per-read MSD and BLD values for every OTU of both regions

## How to apply

Load the HMP-derived V1V3 and V3V5 OTU member sequences together with each OTU's representative sequence. Place member reads onto a fixed reference phylogeny using pplacer/SEPP to obtain the placement-induced multiple sequence alignment for each OTU. From that placed alignment, compute MSD between the representative sequence and every remaining read (values fall in the 0-1 dissimilarity range, compared against the 0.03 threshold). Estimate a maximum-likelihood phylogeny for each OTU alignment with FastTree 2 and read BLD off the representative-to-member path lengths on that tree. Aggregate per-read MSD and BLD values across all OTUs of both hypervariable regions into msd_bld_dissimilarities.csv.

## Related tools

- **pplacer** (Maximum-likelihood phylogenetic placement of OTU member reads onto a fixed reference tree to induce the per-OTU MSA) — github.com/namphuon/phylo
- **SEPP** (SATé-enabled phylogenetic placement used for placing reads onto the reference phylogeny) — github.com/namphuon/phylo
- **FastTree 2** (Estimates the approximate maximum-likelihood phylogeny for each OTU alignment from which BLD values are computed) — github.com/namphuon/phylo
- **PASTA** (Ultra-large multiple sequence alignment tool used in the alignment/phylogeny pipeline supporting OTU alignments) — github.com/namphuon/phylo

## Examples

```
FastTree2 -nt otu_placed_aln.fasta > otu_ml.nwk
```

## Evaluation signals

- Recompute summary aggregates from msd_bld_dissimilarities.csv and verify they reproduce the reported statistic that 80.5% of V3V5 OTUs contain at least one read with MSD>0.03 from the representative
- Check all per-read MSD values fall within the 0-1 dissimilarity range
- Verify every read in each V1V3/V3V5 OTU has both an MSD and a BLD entry relative to its representative sequence (no missing members)
- Confirm the ML tree used for BLD was estimated per-OTU on that OTU's placement-induced alignment (FastTree 2), not on a global tree

## Limitations

- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation, so MSD/BLD often exceed the 0.03 threshold even inside one OTU — a poorly clustered OTU, not a pipeline error
- OTUs are annotated based on a single representative sequence, so the metrics expose downstream annotation risk but do not correct the original PSD clustering
- MSD depends on the placement-induced MSA quality from pplacer/SEPP; errors in read placement propagate into both MSD and BLD

## Evidence

- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [abstract] V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree: "V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree"
