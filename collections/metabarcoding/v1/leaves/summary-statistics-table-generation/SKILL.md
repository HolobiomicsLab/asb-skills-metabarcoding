---
name: summary-statistics-table-generation
description: Use when evaluating how compact existing 97% sequence-similarity OTUs
  (e.g. HMP V1V3/V3V5) are by computing MSD/BLD from each representative to member
  sequences and aggregating percentages above the 0.03 threshold.
when_to_use_negative:
- Input is already a feature/OTU abundance table with no underlying sequences — MSD
  and BLD cannot be computed without the member sequences.
- The OTUs were clustered with a phylogeny-aware method rather than 97% percent sequence
  similarity, so re-measuring PSD-cluster compactness is not the goal.
- The analysis aims to re-cluster sequences into new OTUs rather than audit the compactness
  of existing 97% PSD clusters.
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0091
tools:
- name: phylo
  role: Package providing the distance routines used to compute MSD between representative
    and member sequences in each OTU
  repo: github.com/namphuon/phylo
- name: SEPP
  role: Phylogenetic placement of OTU sequences onto the reference tree to enable
    BLD computation
- name: pplacer
  role: Maximum-likelihood phylogenetic placement of sequences onto a fixed reference
    tree used for BLD
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:fbe2c60918860a86c683b2fe43197513e369a8d4d1a317dac8ab88b43d9fac25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - SEPP
  - phylo
  - pplacer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 6
---

# summary-statistics-table-generation

## Summary

Generates a region-level compactness summary table for 16S rRNA OTUs by re-measuring 97% PSD clusters (e.g. HMP V1V3 and V3V5) with MSD and BLD against each OTU's representative sequence, then averaging and thresholding the distances at 0.03.

## When to use

Use when evaluating how compact existing 97% sequence-similarity OTUs (e.g. HMP V1V3/V3V5) are by computing MSD/BLD from each representative to member sequences and aggregating percentages above the 0.03 threshold.

## When NOT to use

- Input is already a feature/OTU abundance table with no underlying sequences — MSD and BLD cannot be computed without the member sequences.
- The OTUs were clustered with a phylogeny-aware method rather than 97% percent sequence similarity, so re-measuring PSD-cluster compactness is not the goal.
- The analysis aims to re-cluster sequences into new OTUs rather than audit the compactness of existing 97% PSD clusters.

## Inputs

- HMP V1V3 OTU sequence collection clustered at 97% PSD similarity (from github.com/namphuon/phylo)
- HMP V3V5 OTU sequence collection clustered at 97% PSD similarity (from github.com/namphuon/phylo)
- Representative sequence per OTU
- Fixed 16S reference tree for phylogenetic placement (SEPP/pplacer)

## Outputs

- msd_bld_summary_table.csv with per-region averages (V1V3: MSD 0.016, BLD 0.054; V3V5: MSD 0.014, BLD 0.049) and threshold percentages
- Per-OTU MSD distances between representative and member sequences
- Per-OTU BLD distances from phylogenetic placement

## How to apply

Obtain the HMP V1V3 and V3V5 OTU sequence collections clustered at the 97% PSD threshold from the github.com/namphuon/phylo repository. For each OTU, compute the MSD between the representative sequence and every remaining member using the phylo package's distance routines, and compute BLD by phylogenetic placement of the OTU sequences onto a reference tree with SEPP/pplacer. Aggregate per region: average MSD, average BLD, percentage of OTUs with at least one sequence MSD > 0.03, percentage of OTUs with average MSD > 0.03, and percentage of sequences exceeding 0.03 for MSD and BLD. The rationale is that PSD at 97% similarity can overestimate evolutionary similarity, so MSD and BLD reveal poorly clustered OTUs whose members are >3% dissimilar from the annotated representative. Write the full summary table to msd_bld_summary_table.csv and compare against the paper's reported values within a small tolerance.

## Related tools

- **phylo** (Package providing the distance routines used to compute MSD between representative and member sequences in each OTU) — github.com/namphuon/phylo
- **SEPP** (Phylogenetic placement of OTU sequences onto the reference tree to enable BLD computation)
- **pplacer** (Maximum-likelihood phylogenetic placement of sequences onto a fixed reference tree used for BLD)

## Examples

```
df.to_csv('msd_bld_summary_table.csv')
```

## Evaluation signals

- V1V3 average MSD ≈ 0.016 and average BLD ≈ 0.054; V3V5 average MSD ≈ 0.014 and average BLD ≈ 0.049, within a small tolerance.
- 84.4% (V1V3) and 80.5% (V3V5) of OTUs have at least one sequence with MSD > 0.03 from the representative.
- 9.5% (V1V3) and 4.7% (V3V5) of OTUs have average MSD > 0.03.
- 6.4% (V1V3) and 2.7% (V3V5) of all sequences have MSD > 0.03; 81.2% (V1V3) and 78.9% (V3V5) of sequences have BLD > 0.03.
- msd_bld_summary_table.csv exists, contains all aggregated percentages and averages for both regions, and matches the paper's reported values.

## Limitations

- Pairwise sequence alignment-based similarity computation can result in poorly clustered OTUs, so PSD-based compactness values are not comparable to MSD/BLD values.
- OTU compactness varies greatly across the taxonomic tree, so region-level averages can mask taxon-specific outliers.
- BLD depends on phylogenetic placement (SEPP/pplacer) onto a fixed reference tree, so results are sensitive to the reference tree choice.
- The phylo repository README provides minimal documentation ('Phylo'), so usage may require inspecting the package source.

## Evidence

- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [results] 2.7% of all sequences across all the V3V5 OTUs had a MSD >3% from the representative: "2.7% of all sequences across all the V3V5 OTUs had a MSD >3% from the representative"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
