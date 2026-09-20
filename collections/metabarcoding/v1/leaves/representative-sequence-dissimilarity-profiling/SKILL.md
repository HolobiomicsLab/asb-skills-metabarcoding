---
name: representative-sequence-dissimilarity-profiling
description: 'Use when auditing OTU compactness: given HMP-style OTUs clustered with
  PSD at a 97% sequence similarity threshold (V1V3 and/or V3V5 16S regions) plus taxonomic
  assignments, to compute per-OTU MSD and BLD and detect taxa (e.g. TM7, Cyanobacteria,
  Lactobacillus, Staphylococcus) whose OTUs exceed a 0.'
when_to_use_negative:
- OTUs were not built with percent sequence dissimilarity (PSD) clustering at the
  97% threshold — compactness re-measurement of representative-vs-member distances
  is not the appropriate audit
- The analysis goal is clustering or de novo OTU generation itself rather than quality
  assessment of existing OTUs; use QIIME/mothur clustering workflows instead
- No representative sequence or per-OTU member memberships are available (e.g. only
  a collapsed feature table), so MSD/BLD against the representative cannot be computed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0637
tools:
- name: phylo
  role: Software package providing the distance routines used to compute MSD (mean
    sequence dissimilarity) and BLD (branch length distance) between OTU representative
    sequences and member sequences
  repo: github.com/namphuon/phylo
- name: QIIME
  role: Pipeline used to generate the HMP V1V3 and V3V5 OTUs that this skill audits
    (PSD-based clustering at a 97% sequence similarity threshold)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:8269759fa40ac74f2802da9e57cb2917d1c621971d41b31c1b7457735c895908
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME
  - phylo
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# representative-sequence-dissimilarity-profiling

## Summary

Quantifies how 'compact' 16S rRNA OTUs are by computing MSD (mean sequence dissimilarity) and BLD (branch length distance) between each OTU's representative sequence and every remaining member sequence, then aggregating these distances per taxon and per 16S hypervariable region (V1V3 vs V3V5) to expose taxon-specific clustering quality of the 97% PSD HMP OTUs.

## When to use

Use when auditing OTU compactness: given HMP-style OTUs clustered with PSD at a 97% sequence similarity threshold (V1V3 and/or V3V5 16S regions) plus taxonomic assignments, to compute per-OTU MSD and BLD and detect taxa (e.g. TM7, Cyanobacteria, Lactobacillus, Staphylococcus) whose OTUs exceed a 0.03 dissimilarity threshold.

## When NOT to use

- OTUs were not built with percent sequence dissimilarity (PSD) clustering at the 97% threshold — compactness re-measurement of representative-vs-member distances is not the appropriate audit
- The analysis goal is clustering or de novo OTU generation itself rather than quality assessment of existing OTUs; use QIIME/mothur clustering workflows instead
- No representative sequence or per-OTU member memberships are available (e.g. only a collapsed feature table), so MSD/BLD against the representative cannot be computed

## Inputs

- HMP V1V3 OTU sequence sets with taxonomic annotations (PSD, 97% similarity threshold)
- HMP V3V5 OTU sequence sets with taxonomic annotations (PSD, 97% similarity threshold)
- Representative sequence and full member sequence sets per OTU
- Reference phylogeny / alignment required for computing BLD (branch length distance)

## Outputs

- taxon_compactness_summary.csv with per-taxon, per-region mean MSD and mean BLD (e.g. TM7 V1V3 0.020/0.171; Cyanobacteria V1V3 0.007/0.023; Staphylococcus V1V3 0.017/0.017)
- Per-OTU MSD and BLD values between representative sequence and each remaining member sequence

## How to apply

Load the HMP V1V3 and V3V5 OTU sequence sets (generated with QIIME using PSD at 97% sequence similarity) together with their taxonomic annotations. For each OTU, compute MSD (mean pairwise sequence dissimilarity) and BLD (branch length distance on the phylogenetic tree) between the single representative sequence and every remaining member sequence using the phylo package's distance routines (github.com/namphuon/phylo). Group OTUs by taxonomic assignment and 16S hypervariable region, and aggregate per-taxon mean MSD and mean BLD, retaining edge cases such as the single-OTU Lactobacillus entry (OTU 7767, MSD 0.050/BLD 0.519). Write the taxon-by-region metrics to a summary table (e.g. taxon_compactness_summary.csv) and compare against the paper's reported values such as TM7 V1V3 mean MSD 0.020 / BLD 0.171 versus V3V5 MSD 0.016 / BLD 0.046. The rationale is that OTUs are annotated solely by their representative sequence, so large MSD/BLD values — often exceeding the 0.03 threshold — reveal that sequences far from the representative were clustered into the same OTU by 97% PSD, with consequences for downstream annotation.

## Related tools

- **phylo** (Software package providing the distance routines used to compute MSD (mean sequence dissimilarity) and BLD (branch length distance) between OTU representative sequences and member sequences) — github.com/namphuon/phylo
- **QIIME** (Pipeline used to generate the HMP V1V3 and V3V5 OTUs that this skill audits (PSD-based clustering at a 97% sequence similarity threshold))

## Evaluation signals

- Computed per-taxon mean MSD and BLD per region match the paper-reported compactness values within tabulated precision (e.g. TM7 V1V3 mean MSD 0.020 and mean BLD 0.171; TM7 V3V5 0.016/0.046; Cyanobacteria V1V3 0.007/0.023 vs V3V5 0.010/0.025)
- TM7 V1V3 mean BLD (0.171) is more than three times the average BLD, consistent with the reported finding of much greater V1V3 diversity for TM7 and much lower V3V5 diversity
- Single-OTU taxa are retained as-is, e.g. Lactobacillus V1V3 mean 0.026/0.118 with OTU 7767 at 0.050/0.519, and Staphylococcus V1V3 at 0.017/0.017
- Output table contains one row per taxon per 16S region (V1V3 and V3V5) with both mean MSD and mean BLD columns populated
- Aggregate statistics reconcile with region-wide results such as 80.5% of V3V5 OTUs having at least one sequence with MSD > 0.03 from the representative

## Limitations

- OTU annotation inherits entirely from a single representative sequence, so poorly clustered (non-compact) OTUs propagate annotation errors to all member sequences
- The traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs; 97% similarity is only a rough approximation for delineating species and can overestimate evolutionary similarity
- Taxa represented by a single OTU (e.g. Lactobacillus OTU 7767) cannot support per-taxon diversity comparisons across regions
- Compactness varies greatly across the taxonomic tree, so region-level or global averages mask taxon-specific behavior such as TM7's high V1V3 BLD

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [abstract] From each OTU, a single sequence is selected as a representative: "From each OTU, a single sequence is selected as a representative"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold: "we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold"
