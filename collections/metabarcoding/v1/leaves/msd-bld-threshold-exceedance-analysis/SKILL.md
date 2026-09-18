---
name: msd-bld-threshold-exceedance-analysis
description: Use when OTUs were clustered at a percent sequence similarity threshold
  (typically 97% PSD) and you need to quantify how poorly clustered they are by re-measuring
  pairwise MSD and BLD from the representative sequence to all members.
when_to_use_negative:
- The sequences have not been clustered into OTUs at a percent sequence similarity
  threshold (e.g. you are working with an unclustered FASTQ/amplicon dataset or de
  novo phylogenetic placement results)
- No reference tree is available for phylogenetic placement, so BLD cannot be computed
  with SEPP/pplacer
- The goal is taxonomic annotation of OTUs rather than assessing OTU compactness/dissimilarity
  from the representative sequence
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0084
tools:
- name: phylo
  role: Package whose distance routines compute MSD and BLD between the representative
    sequence and every remaining member sequence of each OTU
  repo: github.com/namphuon/phylo
- name: SEPP
  role: SATé-enabled phylogenetic placement used to place OTU sequences onto a reference
    tree for BLD computation
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of sequences onto a fixed reference
    tree, underlying the BLD step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:978c36f0e45cf0bbb1f81bacb4d6f782bc21c9ae80185ed72b27b2378920597f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - SEPP
  - phylo
  - pplacer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# msd-bld-threshold-exceedance-analysis

## Summary

Re-measures the compactness of 97% PSD-clustered 16S rRNA OTUs (HMP V1V3 and V3V5 regions) using two dissimilarity metrics — multiple sequence dissimilarity (MSD) and branch length dissimilarity (BLD) — between each representative sequence and all member sequences, and quantifies how often the 0.03 dissimilarity threshold is exceeded.

## When to use

Use when OTUs were clustered at a percent sequence similarity threshold (typically 97% PSD) and you need to quantify how poorly clustered they are by re-measuring pairwise MSD and BLD from the representative sequence to all members.

## When NOT to use

- The sequences have not been clustered into OTUs at a percent sequence similarity threshold (e.g. you are working with an unclustered FASTQ/amplicon dataset or de novo phylogenetic placement results)
- No reference tree is available for phylogenetic placement, so BLD cannot be computed with SEPP/pplacer
- The goal is taxonomic annotation of OTUs rather than assessing OTU compactness/dissimilarity from the representative sequence

## Inputs

- HMP V1V3 OTU sequence collections clustered at 97% PSD similarity (from github.com/namphuon/phylo)
- HMP V3V5 OTU sequence collections clustered at 97% PSD similarity (from github.com/namphuon/phylo)
- A fixed reference tree for BLD computation via phylogenetic placement

## Outputs

- Per-region average MSD and average BLD values (V1V3: 0.016 / 0.054; V3V5: 0.014 / 0.049)
- Aggregated percentage table: OTUs with at least one sequence > 0.03 (84.4% V1V3 / 80.5% V3V5 MSD), OTUs with average MSD > 0.03 (9.5% / 4.7%), sequences > 0.03 MSD (6.4% / 2.7%), sequences > 0.03 BLD (81.2% / 78.9%)
- msd_bld_summary_table.csv containing the full summary statistics

## How to apply

Obtain the HMP V1V3 and V3V5 OTU sequence collections clustered at the 97% PSD threshold from the github.com/namphuon/phylo repository. For every OTU, compute MSD between the representative sequence and every remaining member using the phylo package's distance routines. Compute BLD by phylogenetic placement of the OTU sequences onto a fixed reference tree with SEPP/pplacer. Aggregate per region the average MSD (V1V3: 0.016; V3V5: 0.014), average BLD (V1V3: 0.054; V3V5: 0.049), and the fractions of OTUs/sequences exceeding the 0.03 dissimilarity threshold, then compare against the paper's reported values and write the full summary to msd_bld_summary_table.csv. The rationale is that pairwise sequence alignment-based similarity computation can result in poorly clustered OTUs, and since OTUs are annotated via a single representative sequence, such poor clustering propagates to downstream analyses.

## Related tools

- **phylo** (Package whose distance routines compute MSD and BLD between the representative sequence and every remaining member sequence of each OTU) — github.com/namphuon/phylo
- **SEPP** (SATé-enabled phylogenetic placement used to place OTU sequences onto a reference tree for BLD computation)
- **pplacer** (Maximum-likelihood phylogenetic placement of sequences onto a fixed reference tree, underlying the BLD step)

## Evaluation signals

- Average MSD/BLD per region must match the paper within a small tolerance: V1V3 average MSD 0.016 and BLD 0.054; V3V5 average MSD 0.014 and BLD 0.049
- Percentage of OTUs with at least one sequence MSD > 0.03 must reproduce 84.4% (V1V3) and 80.5% (V3V5)
- Percentage of OTUs with average MSD > 0.03 must reproduce 9.5% (V1V3) and 4.7% (V3V5)
- Percentage of sequences exceeding 0.03 must reproduce 6.4% / 2.7% for MSD and 81.2% / 78.9% for BLD
- The summary table is written to msd_bld_summary_table.csv with all per-region aggregates present

## Limitations

- PSD (percent sequence similarity) can overestimate evolutionary similarity between pairs of sequences, which is precisely why MSD/BLD re-measurement reveals poorly clustered OTUs
- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation
- OTU compactness varies greatly across the taxonomic tree, so single region-wide averages can mask large variation between individual OTUs

## Evidence

- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [results] 2.7% of all sequences across V3V5 OTUs had a MSD >3% from the representative: "2.7% of all sequences across all the V3V5 OTUs had a MSD >3% from the representative"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] We will show that the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU.: "We will show that the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU."
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
