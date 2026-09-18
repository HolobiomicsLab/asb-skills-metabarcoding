---
name: taxonomic-group-aggregation-of-distances
description: Use when OTUs were clustered at a 97% percent sequence dissimilarity
  (PSD) threshold and you need to characterize, per taxonomic group and 16S region,
  how evolutionarily compact those OTUs are using MSD and BLD rather than the original
  clustering similarity.
when_to_use_negative:
- The OTUs were clustered with a method other than percent sequence dissimilarity
  (PSD) at 97%, since the compactness reinterpretation specifically targets PSD-based
  OTUs
- The analysis goal is species delineation itself rather than auditing compactness
  of already-built OTUs
- Only a feature/abundance table is available without the underlying OTU member sequences,
  since MSD and BLD require pairwise sequence and phylogenetic distances
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0084
tools:
- name: QIIME
  role: Pipeline that generated the HMP V1V3 and V3V5 OTUs using PSD at a 97% sequence
    similarity threshold, producing the input OTU sequence sets
- name: phylo
  role: Repository providing the distance routines used to compute MSD and BLD between
    representative sequences and remaining OTU members
  repo: https://github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:7a8a3e8131d25c5293cbdb90a462088f2a4e3d6d149bc38db1e24793f1fffdcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - phylo
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# Taxonomic-group aggregation of OTU compactness distances

## Summary

Aggregates per-OTU MSD (mean sequence dissimilarity) and BLD (branch length distance) values, computed between each OTU representative sequence and its remaining members, into per-taxon mean metrics within each 16S hypervariable region (V1V3 vs V3V5) to quantify how OTU compactness varies across the taxonomic tree.

## When to use

Use when OTUs were clustered at a 97% percent sequence dissimilarity (PSD) threshold and you need to characterize, per taxonomic group and 16S region, how evolutionarily compact those OTUs are using MSD and BLD rather than the original clustering similarity.

## When NOT to use

- The OTUs were clustered with a method other than percent sequence dissimilarity (PSD) at 97%, since the compactness reinterpretation specifically targets PSD-based OTUs
- The analysis goal is species delineation itself rather than auditing compactness of already-built OTUs
- Only a feature/abundance table is available without the underlying OTU member sequences, since MSD and BLD require pairwise sequence and phylogenetic distances

## Inputs

- HMP V1V3 OTU sequence set with taxonomic annotations (QIIME/PSD, 97% similarity threshold)
- HMP V3V5 OTU sequence set with taxonomic annotations (QIIME/PSD, 97% similarity threshold)
- OTU representative-to-member sequence assignments
- Taxonomic assignment per OTU (representative sequence annotation inherited by all members)

## Outputs

- Per-taxon, per-region aggregated mean MSD values (e.g. taxon_compactness_summary.csv)
- Per-taxon, per-region aggregated mean BLD values
- Per-OTU MSD/BLD pairs between representative and member sequences

## How to apply

Load the HMP-derived V1V3 and V3V5 OTU sequence sets (generated with QIIME using PSD at a 97% sequence similarity threshold) with their taxonomic annotations from the phylo repository data. Group OTU member sequences by taxonomic assignment, isolating taxa of interest (e.g. TM7, Cyanobacteria, Lactobacillus, Staphylococcus, Sneathia) within each 16S hypervariable region. For each OTU, compute the MSD and BLD between the representative sequence and every remaining member sequence using the phylo package's distance routines, then average these per-OTU values within each taxon-by-region cell, retaining exceptional entries such as the single-OTU Lactobacillus 7767 (MSD 0.050, BLD 0.519). Compare the aggregated means against the 0.03 MSD threshold highlighted in the paper: for example TM7 V1V3 shows mean MSD 0.020 and BLD 0.171 (more than three times the average BLD) versus much lower V3V5 diversity (MSD 0.016, BLD 0.046). Write the taxon-by-region metrics to a summary table (e.g. taxon_compactness_summary.csv) for validation against paper-reported values.

## Related tools

- **QIIME** (Pipeline that generated the HMP V1V3 and V3V5 OTUs using PSD at a 97% sequence similarity threshold, producing the input OTU sequence sets)
- **phylo** (Repository providing the distance routines used to compute MSD and BLD between representative sequences and remaining OTU members) — https://github.com/namphuon/phylo

## Examples

```
python compute_taxon_compactness.py --otu-dir hmp_otus --regions V1V3,V3V5 --metrics MSD,BLD --out taxon_compactness_summary.csv
```

## Evaluation signals

- Computed mean MSD and BLD per taxon per region match paper-reported compactness values within tabulated precision (e.g. TM7 V1V3 mean MSD 0.020 and BLD 0.171; V3V5 MSD 0.016 and BLD 0.046)
- Cyanobacteria aggregates reproduce V1V3 0.007/0.023 vs V3V5 0.010/0.025
- Staphylococcus V1V3 mean MSD 0.017 and BLD 0.017 reproduced exactly
- Lactobacillus V1V3 aggregate 0.026/0.118 is retained alongside the single-OTU OTU 7767 entry at 0.050/0.519 rather than lost during aggregation
- Every MSD and BLD value is computed between the OTU representative sequence and all remaining member sequences, not among arbitrary pairs

## Limitations

- Percent sequence similarity (PSD) can overestimate evolutionary similarity, so OTUs built at 97% may be poorly clustered and both MSD and BLD often exceed the 0.03 threshold
- Because OTUs are annotated based on a single representative sequence, poorly clustered OTUs propagate annotation error to all members
- Taxa represented by a single OTU (e.g. Lactobacillus OTU 7767) cannot be aggregated meaningfully and must be reported as per-OTU values

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] We will show that the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU.: "We will show that the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU."
- [intro] we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold: "we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold"
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [results] the candidate phylum TM7 showed much greater diversity in the V1V3 region, with a mean MSD and BLD of 0.020 and 0.171, more than three times the average BLD: [evidence span withheld — anchor status: not_found]
