---
name: 16s-otu-compactness-analysis
description: Use when you have OTUs built at a 97% percent sequence similarity threshold
  and need to quantify how poorly clustered they are using MSD and BLD metrics, either
  across 16S hypervariable regions or across taxonomic groups.
when_to_use_negative:
- When the analysis goal is de novo OTU clustering itself — this skill re-measures
  compactness of already-clustered OTUs, it does not cluster sequences
- When input sequences are not OTUs built from 16S amplicon data at a percent sequence
  similarity threshold (e.g. shotgun metagenomic bins or protein families)
- When only pairwise PSD similarity is needed — this skill specifically evaluates
  PSD-based OTUs with the alternative MSD and BLD metrics
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
tools:
- name: phylo
  role: Package providing the distance routines used to compute MSD (and supporting
    BLD computation) between representative and member sequences within each OTU
  repo: github.com/namphuon/phylo
- name: SEPP
  role: Phylogenetic placement of OTU sequences onto a reference tree to enable BLD
    computation
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of sequences onto a fixed reference
    tree for branch length distance calculation
- name: QIIME
  role: Pipeline used upstream to generate the HMP OTUs at the 97% PSD sequence similarity
    threshold
merged_aliases:
- 16s-otu-compactness-assessment
merged_alias_records:
- alias: 16s-otu-compactness-assessment
  slug: 16s-otu-compactness-assessment
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:725ddcc98a165048c25a90b369d6b089f503ad4197a6258bc37f24122cb53c8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - QIIME
  - SEPP
  - phylo
  - pplacer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# 16s-otu-compactness-analysis

## Summary

Re-measures the compactness of 16S rRNA OTUs that were clustered at 97% pairwise sequence dissimilarity (PSD) by computing Mean Sequence Dissimilarity (MSD) and Branch Length Distance (BLD) between each OTU's representative sequence and all remaining member sequences, both per-region (HMP V1V3 vs V3V5) and per-taxon.

## When to use

Use when you have OTUs built at a 97% percent sequence similarity threshold and need to quantify how poorly clustered they are using MSD and BLD metrics, either across 16S hypervariable regions or across taxonomic groups.

## When NOT to use

- When the analysis goal is de novo OTU clustering itself — this skill re-measures compactness of already-clustered OTUs, it does not cluster sequences
- When input sequences are not OTUs built from 16S amplicon data at a percent sequence similarity threshold (e.g. shotgun metagenomic bins or protein families)
- When only pairwise PSD similarity is needed — this skill specifically evaluates PSD-based OTUs with the alternative MSD and BLD metrics

## Inputs

- OTU sequence collections clustered at 97% PSD threshold with taxonomic annotations (e.g. HMP V1V3 and V3V5 OTUs from the phylo repository)
- OTU representative sequences and their member sequences
- A fixed 16S reference tree for phylogenetic placement

## Outputs

- Region-level summary table of average MSD, average BLD, and percentages of OTUs/sequences exceeding 0.03 for MSD and BLD (e.g. msd_bld_summary_table.csv)
- Taxon-by-region compactness metrics (mean MSD and mean BLD per taxon per region, e.g. taxon_compactness_summary.csv)

## How to apply

Load the OTU sequence collections (e.g. HMP V1V3 and V3V5 OTUs clustered with QIIME using PSD at a 97% sequence similarity threshold) from github.com/namphuon/phylo. For each OTU, compute the MSD (mean sequence dissimilarity) between the representative sequence and every remaining member sequence using the phylo package's distance routines; compute BLD by phylogenetic placement of the OTU sequences onto a fixed reference tree with SEPP/pplacer. Aggregate results per region (V1V3, V3V5) and optionally per taxon (e.g. TM7, Cyanobacteria, Lactobacillus, Staphylococcus), reporting average MSD, average BLD, and the fraction of OTUs/sequences exceeding the 0.03 (3% dissimilarity) threshold. The rationale is that PSD-based clustering can overestimate evolutionary similarity, so poorly clustered OTUs matter because OTUs are annotated based on a single representative sequence.

## Related tools

- **phylo** (Package providing the distance routines used to compute MSD (and supporting BLD computation) between representative and member sequences within each OTU) — github.com/namphuon/phylo
- **SEPP** (Phylogenetic placement of OTU sequences onto a reference tree to enable BLD computation)
- **pplacer** (Maximum-likelihood phylogenetic placement of sequences onto a fixed reference tree for branch length distance calculation)
- **QIIME** (Pipeline used upstream to generate the HMP OTUs at the 97% PSD sequence similarity threshold)

## Examples

```
python scripts/compute_msd_bld.py --region V1V3 --region V3V5 --threshold 0.03 --output msd_bld_summary_table.csv
```

## Evaluation signals

- Computed region-level averages match reported values: V1V3 average MSD 0.016 and average BLD 0.054; V3V5 average MSD 0.014 and average BLD 0.049
- Computed percentages match reported values within a small tolerance: 84.4%/80.5% of V1V3/V3V5 OTUs with at least one sequence MSD > 0.03; 9.5%/4.7% of OTUs with average MSD > 0.03; 6.4%/2.7% of sequences with MSD > 0.03; 81.2%/78.9% of sequences with BLD > 0.03
- Per-taxon values match paper-reported compactness: TM7 V1V3 mean MSD 0.020 and mean BLD 0.171 (more than three times the average BLD) vs TM7 V3V5 MSD 0.016/BLD 0.046
- Single-OTU taxa are retained correctly (e.g. Lactobacillus OTU 7767 at MSD 0.050/BLD 0.519; Staphylococcus V1V3 0.017/0.017)
- Output CSV contains one row per taxon per region with mean MSD and mean BLD columns populated at the tabulated precision

## Limitations

- PSD-based 97% similarity is only a rough approximation for delineating species, so the OTUs being evaluated are themselves known to potentially overestimate evolutionary similarity
- BLD requires phylogenetic placement (SEPP/pplacer) onto a fixed reference tree, adding a dependency and potential placement error
- Results can be sparse for rare taxa — some taxa are represented by only a single OTU (e.g. Lactobacillus OTU 7767), limiting per-taxon aggregation
- The repository README provides no usage documentation, so reproducibility depends on the paper's methods rather than package docs

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
