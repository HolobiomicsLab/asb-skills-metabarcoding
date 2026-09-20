---
name: phylogenetic-distance-computation
description: Use when you have 16S OTUs (e.g. HMP V1V3/V3V5 clustered at 97% PSD with
  QIIME) and need to quantify how compact each OTU is under MSD and BLD rather than
  the clustering threshold metric.
when_to_use_negative:
- When the goal is simply to cluster sequences into OTUs de novo — this skill re-measures
  existing PSD-clustered OTUs rather than performing the clustering itself
- When OTUs are already defined by a phylogeny-aware method and their evolutionary
  compactness under MSD/BLD is not in question
- When only a single representative sequence per OTU is available without the full
  member sequence sets — MSD and BLD between representative and members cannot be
  computed
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0084
tools:
- name: SEPP
  role: Phylogenetic placement of OTU member sequences onto a fixed reference tree
    to derive BLD
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of sequences onto a fixed reference
    tree for BLD computation
- name: QIIME
  role: Pipeline that generated the HMP OTUs at the 97% PSD threshold that this skill
    re-evaluates
- name: phylo
  role: Package providing the MSD/BLD distance routines applied between representative
    and member sequences
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:f9ee8fce32b45c49663fd1da328450cde7c37c11134c94b0da98d64560ba77a6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME
  - SEPP
  - phylo
  - pplacer
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# phylogenetic-distance-computation

## Summary

Re-measures compactness of OTUs clustered at 97% percent sequence dissimilarity (PSD) using two alternative dissimilarity metrics — mean sequence dissimilarity (MSD) and branch length distance (BLD) — between each OTU's representative sequence and all member sequences, exposing poorly clustered OTUs that PSD hides.

## When to use

Use when you have 16S OTUs (e.g. HMP V1V3/V3V5 clustered at 97% PSD with QIIME) and need to quantify how compact each OTU is under MSD and BLD rather than the clustering threshold metric.

## When NOT to use

- When the goal is simply to cluster sequences into OTUs de novo — this skill re-measures existing PSD-clustered OTUs rather than performing the clustering itself
- When OTUs are already defined by a phylogeny-aware method and their evolutionary compactness under MSD/BLD is not in question
- When only a single representative sequence per OTU is available without the full member sequence sets — MSD and BLD between representative and members cannot be computed

## Inputs

- OTU sequence collections clustered at 97% PSD (e.g. HMP V1V3 and V3V5 OTUs from github.com/namphuon/phylo)
- Taxonomic annotations for OTU member sequences (for per-taxon aggregation, e.g. TM7, Cyanobacteria, Lactobacillus, Staphylococcus, Sneathia)
- Reference phylogenetic tree for phylogenetic placement (SEPP/pplacer)

## Outputs

- Per-OTU MSD and BLD values between representative sequence and each member sequence
- Aggregated summary tables, e.g. msd_bld_summary_table.csv (mean MSD/BLD per region and percentages of OTUs/sequences with distance > 0.03) and taxon_compactness_summary.csv (taxon-by-region mean MSD/BLD)

## How to apply

Load the OTU sequence collections (e.g. HMP V1V3 and V3V5 OTUs generated at the 97% PSD threshold from github.com/namphuon/phylo). For each OTU, compute the MSD between the representative sequence and every remaining member sequence using the phylo package's distance routines. Compute BLD by phylogenetic placement of the OTU member sequences onto a fixed reference tree with SEPP/pplacer, then reading branch length distances from representative to members. Aggregate per region or per taxon (e.g. mean MSD/BLD per taxon per 16S hypervariable region), writing results such as msd_bld_summary_table.csv or taxon_compactness_summary.csv. The rationale is that pairwise percent sequence similarity can overestimate evolutionary similarity, so OTUs that pass a 97% PSD cutoff can still contain sequences >3% (MSD>0.03) dissimilar or far away in branch length from the representative that supplies the OTU's annotation.

## Related tools

- **SEPP** (Phylogenetic placement of OTU member sequences onto a fixed reference tree to derive BLD)
- **pplacer** (Maximum-likelihood phylogenetic placement of sequences onto a fixed reference tree for BLD computation)
- **QIIME** (Pipeline that generated the HMP OTUs at the 97% PSD threshold that this skill re-evaluates)
- **phylo** (Package providing the MSD/BLD distance routines applied between representative and member sequences) — github.com/namphuon/phylo

## Examples

```
python compute_distances.py --otu-dir hmp_v1v3_otus --metric MSD --metric BLD --threshold 0.03 --out msd_bld_summary_table.csv
```

## Evaluation signals

- Computed percentages must match reported values within tolerance: e.g. 84.4%/80.5% of V1V3/V3V5 OTUs with at least one sequence MSD > 0.03; 9.5%/4.7% of OTUs with average MSD > 0.03; 6.4%/2.7% of sequences with MSD > 0.03; 81.2%/78.9% of sequences with BLD > 0.03
- Average MSD/BLD per region must match reported values: V1V3 MSD 0.016 / BLD 0.054; V3V5 MSD 0.014 / BLD 0.049
- Per-taxon means must match tabulated precision, e.g. TM7 V1V3 MSD 0.020 / BLD 0.171 vs V3V5 MSD 0.016 / BLD 0.046; Cyanobacteria V1V3 0.007/0.023 vs V3V5 0.010/0.025; Lactobacillus V1V3 0.026/0.118 with OTU 7767 at 0.050/0.519; Staphylococcus V1V3 0.017/0.017
- Output schema check: summary tables (msd_bld_summary_table.csv, taxon_compactness_summary.csv) contain one row per region or per taxon-by-region combination with both mean MSD and mean BLD columns
- Every OTU member sequence must be covered by exactly one representative-to-member distance computation (MSD and BLD), with no sequences dropped from aggregation

## Limitations

- 97% 16S rRNA sequence similarity is only a rough approximation for delineating species, so PSD-based OTU compactness conclusions do not transfer to taxonomic novelty claims
- Percent sequence similarity can overestimate evolutionary similarity between pairs of sequences, which is the failure mode this skill is designed to detect but also a caveat when interpreting MSD/BLD against the 0.03 threshold
- OTUs are typically annotated based upon a single representative sequence, so poorly clustered OTUs propagate annotation errors to downstream analyses regardless of the distance metrics used

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [intro] the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU: "the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU"
