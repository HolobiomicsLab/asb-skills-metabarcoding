---
name: phylogenetic-sequence-placement
description: Use when individual short 16S reads or OTU member sequences must be assigned
  lineages via phylogenetic placement on a reference tree, or when BLD between representative
  and member sequences must be computed for HMP-style V1V3/V3V5 OTUs.
when_to_use_negative:
- When the goal is only percent-sequence-similarity (PSD) clustering itself — this
  skill annotates/places already-clustered OTU reads, it does not regenerate OTUs
- When input is a pre-computed feature table or OTU abundance matrix with no underlying
  sequence collection — placement requires the actual amplicon sequences
- When full-length 16S sequences can be aligned de novo — SEPP is specifically designed
  for inserting short fragments into a fixed reference tree
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0194
tools:
- name: SEPP
  role: SATé-enabled phylogenetic placement used to insert each short 16S amplicon
    read into the fixed reference tree
- name: pplacer
  role: Linear-time maximum-likelihood/Bayesian placement engine assigning reads and
    OTU sequences to branches of the fixed reference tree
- name: RAxML version 8
  role: Constructs the fixed 16S reference tree used as the placement backbone
- name: PASTA
  role: Builds the ultra-large 16S reference multiple sequence alignment underlying
    the reference tree
- name: FastTree 2
  role: Part of the reference-tree construction lineage supporting the fixed reference
    tree
- name: QIIME
  role: Source of the HMP V1V3/V3V5 OTU read collections clustered by PSD at 97% similarity
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:89ff11973398d71148f3b77e3f038e50074e57e2a7569dfd73907ff8a8d7ddfc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastTree 2
  - PASTA
  - QIIME
  - RAxML version 8
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# phylogenetic-sequence-placement

## Summary

Places 16S rRNA amplicon reads or OTU member sequences onto a fixed reference tree using SEPP and pplacer, enabling per-read taxonomic lineage assignment and branch-length-divergence (BLD) computation instead of inheriting the single representative sequence's annotation.

## When to use

Use when individual short 16S reads or OTU member sequences must be assigned lineages via phylogenetic placement on a reference tree, or when BLD between representative and member sequences must be computed for HMP-style V1V3/V3V5 OTUs.

## When NOT to use

- When the goal is only percent-sequence-similarity (PSD) clustering itself — this skill annotates/places already-clustered OTU reads, it does not regenerate OTUs
- When input is a pre-computed feature table or OTU abundance matrix with no underlying sequence collection — placement requires the actual amplicon sequences
- When full-length 16S sequences can be aligned de novo — SEPP is specifically designed for inserting short fragments into a fixed reference tree

## Inputs

- QIIME-generated HMP OTU read collections for V1V3 and V3V5 clustered by PSD at a 97% sequence similarity threshold
- Fixed 16S reference tree and reference alignment built with RAxML version 8 / PASTA / FastTree 2
- Amplicon FASTA reads or OTU member sequences to be placed
- OTU representative sequences whose inherited annotations serve as the discordance baseline

## Outputs

- Per-read placement table (per_read_placements.csv) mapping every read to its placed lineage/taxonomic label from the reference tree
- Discordance flags identifying reads whose placement-derived taxonomy differs from the OTU representative's annotation
- BLD values between representative and member sequences for OTU compactness evaluation (e.g. average BLD 0.054 for V1V3, 0.049 for V3V5)
- msd_bld_summary_table.csv aggregating per-region MSD/BLD averages and percentage of sequences exceeding the 0.03 threshold

## How to apply

First construct a fixed 16S reference tree and alignment using the RAxML version 8 / PASTA / FastTree 2 lineage of reference-tree construction recommended in the paper. For per-read placement, run SEPP (SATé-enabled phylogenetic placement) to insert each short amplicon read into the reference tree, then use pplacer's linear-time maximum-likelihood/Bayesian placement of sequences onto the fixed reference tree as the placement engine. Extract the placed lineage/taxonomic label for every read and compare it against the annotation the read would inherit from its OTU's single representative sequence; the paper reports 12.9% of V3V5 and 19.8% of V1V3 OTUs contain discordant reads. For OTU compactness assessment, place OTU sequences onto the reference tree with SEPP/pplacer so that BLD can be computed between the representative sequence and every remaining member sequence, complementing MSD measured with the phylo package's distance routines.

## Related tools

- **SEPP** (SATé-enabled phylogenetic placement used to insert each short 16S amplicon read into the fixed reference tree)
- **pplacer** (Linear-time maximum-likelihood/Bayesian placement engine assigning reads and OTU sequences to branches of the fixed reference tree)
- **RAxML version 8** (Constructs the fixed 16S reference tree used as the placement backbone)
- **PASTA** (Builds the ultra-large 16S reference multiple sequence alignment underlying the reference tree)
- **FastTree 2** (Part of the reference-tree construction lineage supporting the fixed reference tree)
- **QIIME** (Source of the HMP V1V3/V3V5 OTU read collections clustered by PSD at 97% similarity)

## Evaluation signals

- Discordance rate must fall in the reported range: 12.9% of V3V5 OTUs and 19.8% of V1V3 OTUs contain reads annotated differently from their representative sequence
- Per-read placement table (per_read_placements.csv) must contain a placement-derived lineage label for every read in the OTU collection
- Average BLD between representatives and members must match reported values (0.054 for V1V3, 0.049 for V3V5) within tolerance
- Percentage of OTUs with at least one sequence >3% dissimilar must match reported values (80.5% for V3V5, 84.4% for V1V3) when combined with MSD computation
- All placements must resolve onto branches of the fixed reference tree — unplaced or failed reads indicate a reference alignment/tree preparation problem

## Limitations

- The 97% PSD threshold used to build the HMP OTUs is only a rough approximation for delineating species, so OTU membership is not guaranteed to be phylogenetically compact
- Representative-sequence annotation mislabels a substantial fraction of OTUs (12.9% V3V5, 19.8% V1V3), motivating per-read placement but also meaning representative-based validation is unreliable
- Placement quality depends on the fixed reference tree and alignment (RAxML/PASTA/FastTree 2 lineage); short V1V3/V3V5 fragments are harder to insert accurately than full-length sequences

## Evidence

- [abstract] V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree: "V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree"
- [abstract] SEPP: SATé-enabled phylogenetic placement: "SEPP: SATé-enabled phylogenetic placement"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
