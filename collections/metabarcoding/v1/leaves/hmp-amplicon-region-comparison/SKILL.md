---
name: hmp-amplicon-region-comparison
description: Use when OTUs were clustered with PSD at a 97% sequence similarity threshold
  from 16S amplicon data (V1V3 or V3V5) and you need to re-measure their compactness
  with MSD/BLD per taxonomic group and per region.
when_to_use_negative:
- Input OTUs were not clustered with percent sequence dissimilarity (PSD) at the ~97%
  threshold — the compactness re-measurement is specifically designed to audit PSD-97%
  OTUs.
- The analysis goal is end-to-end community profiling (abundance tables, beta diversity)
  rather than auditing OTU cluster quality via representative-to-member dissimilarity.
- Sequences lack taxonomic assignments, since the per-taxon aggregation (TM7, Cyanobacteria,
  Lactobacillus, Staphylococcus, Sneathia) requires annotated OTU members.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: QIIME
  role: Pipeline that produced the HMP V1V3/V3V5 OTUs clustered with PSD at a 97%
    sequence similarity threshold
- name: phylo (namphuon/phylo)
  role: Provides the MSD and BLD distance routines computed between each OTU representative
    sequence and its member sequences
  repo: github.com/namphuon/phylo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:ad7bae66bcca0bbdc5a3cb63c69d118a741fff121f7065420ef6afe346a1f0da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - phylo (namphuon/phylo)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# hmp-amplicon-region-comparison

## Summary

Quantifies OTU compactness by computing MSD (mean sequence dissimilarity) and BLD (branch length distance) between each HMP V1V3/V3V5 OTU representative and its member sequences, then aggregates per-taxon mean metrics across the two 16S hypervariable regions.

## When to use

Use when OTUs were clustered with PSD at a 97% sequence similarity threshold from 16S amplicon data (V1V3 or V3V5) and you need to re-measure their compactness with MSD/BLD per taxonomic group and per region.

## When NOT to use

- Input OTUs were not clustered with percent sequence dissimilarity (PSD) at the ~97% threshold — the compactness re-measurement is specifically designed to audit PSD-97% OTUs.
- The analysis goal is end-to-end community profiling (abundance tables, beta diversity) rather than auditing OTU cluster quality via representative-to-member dissimilarity.
- Sequences lack taxonomic assignments, since the per-taxon aggregation (TM7, Cyanobacteria, Lactobacillus, Staphylococcus, Sneathia) requires annotated OTU members.

## Inputs

- HMP V1V3 OTU sequence sets with taxonomic annotations (QIIME, PSD, 97% similarity threshold)
- HMP V3V5 OTU sequence sets with taxonomic annotations (QIIME, PSD, 97% similarity threshold)

## Outputs

- taxon_compactness_summary.csv — per-taxon, per-region mean MSD and mean BLD (e.g. TM7 V1V3 MSD 0.020/BLD 0.171 vs V3V5 MSD 0.016/BLD 0.046; Cyanobacteria V1V3 0.007/0.023 vs V3V5 0.010/0.025; Lactobacillus V1V3 0.026/0.118 with OTU 7767 at 0.050/0.519; Staphylococcus V1V3 0.017/0.017)

## How to apply

Load the HMP V1V3 and V3V5 OTU sequence sets with taxonomic annotations (generated with QIIME using PSD at a 97% sequence similarity threshold). Group OTU member sequences by taxonomic assignment, isolating taxa of interest (e.g. TM7, Cyanobacteria, Lactobacillus, Staphylococcus, Sneathia) within each 16S hypervariable region. For each OTU, compute the MSD (mean sequence dissimilarity) and BLD (branch length distance) between the representative sequence and every remaining member sequence using the phylo package's distance routines. Aggregate per-taxon mean MSD and mean BLD values within each region, retaining single-OTU entries (e.g. Lactobacillus OTU 7767) separately, and write the taxon-by-region table to taxon_compactness_summary.csv. Rationale: 97% percent sequence similarity can overestimate evolutionary similarity, and compactness varies greatly across the taxonomic tree, with both MSD and BLD often exceeding the 0.03 OTU threshold.

## Related tools

- **QIIME** (Pipeline that produced the HMP V1V3/V3V5 OTUs clustered with PSD at a 97% sequence similarity threshold)
- **phylo (namphuon/phylo)** (Provides the MSD and BLD distance routines computed between each OTU representative sequence and its member sequences) — github.com/namphuon/phylo

## Evaluation signals

- TM7 V1V3 mean MSD and BLD reproduce the paper-reported values 0.020 and 0.171 (more than three times the average BLD) within tabulated precision.
- TM7 V3V5 values fall to MSD 0.016 / BLD 0.046, confirming the region-dependent diversity pattern.
- Cyanobacteria V1V3 (MSD 0.007 / BLD 0.023) and V3V5 (MSD 0.010 / BLD 0.025) match within precision.
- Lactobacillus V1V3 aggregates to 0.026/0.118 with the single OTU 7767 retained separately at 0.050/0.519.
- Staphylococcus V1V3 mean MSD and BLD both equal 0.017; output CSV contains one row per taxon per region.

## Limitations

- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation, and percent sequence similarity can overestimate evolutionary similarity between pairs of sequences.
- Compactness varies greatly across the taxonomic tree — both MSD and BLD often exceed the 0.03 threshold for an OTU — so results are taxon- and region-specific and cannot be generalized from one group.
- Single-OTU taxa (e.g. Lactobacillus OTU 7767 at MSD 0.050 / BLD 0.519) produce outlier values that must be retained as separate entries rather than averaged away.
- Poorly clustered OTUs can significantly impact downstream analyses because OTUs are annotated based on a single representative sequence.

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [intro] compactness of OTUs varies across the taxonomic tree and exceeds 0.03: "the compactness of the OTUs varies greatly across the taxonomic tree, and both MSD and BLD often exceed the 0.03 threshold for an OTU."
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] HMP OTUs were generated with QIIME using PSD at 97% similarity: "we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold"
