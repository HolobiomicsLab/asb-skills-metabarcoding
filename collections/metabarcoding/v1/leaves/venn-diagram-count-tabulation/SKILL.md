---
name: venn-diagram-count-tabulation
description: Use when a paper reports species/taxon occurrence counts only as Venn-diagram
  figures or in-text segment counts (per group and shared segments) and the goal is
  to reconstruct per-location/per-source occurrence counts as a tidy CSV.
when_to_use_negative:
- When the underlying OTU/ASV abundance table (e.g., a BIOM or feature table) is available
  — recompute counts directly rather than inferring them from diagram segments
- When raw sequencing data must be processed (adapter trimming with Scythe/Sickle,
  SILVAngs clustering) — this skill assumes counts are already published
- When the question requires quantitative abundance rather than presence/absence species
  occurrence counts
- ''
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:9d75b891a6df788382e5988cbbd9dc8811ef0d4274d0402b245bf4b36e59446b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 6
---

# venn-diagram-count-tabulation

## Summary

A manual expert-review skill for reconstructing species (or taxon) occurrence counts from published Venn-diagram figures, converting reported segment counts (e.g., B-I 774, N-I 669, G-I 548, B-P 882, N-P 553, G-P 415) into a tidy, machine-readable table reconciled against reported source totals.

## When to use

Use when a paper reports species/taxon occurrence counts only as Venn-diagram figures or in-text segment counts (per group and shared segments) and the goal is to reconstruct per-location/per-source occurrence counts as a tidy CSV.

## When NOT to use

- When the underlying OTU/ASV abundance table (e.g., a BIOM or feature table) is available — recompute counts directly rather than inferring them from diagram segments
- When raw sequencing data must be processed (adapter trimming with Scythe/Sickle, SILVAngs clustering) — this skill assumes counts are already published
- When the question requires quantitative abundance rather than presence/absence species occurrence counts
- 

## Inputs

- Published Venn-diagram figures or in-text segment counts (e.g., B-I, N-I, G-I, B-P, N-P, G-P species counts) from the trout-farm metabarcoding article
- Reported source totals used as reconciliation targets (1074 I-source species, 1006 P-source species)

## Outputs

- venn_species_occurrence_counts.csv — tidy table with columns: source type (inflow/pond), location (B, N, G), and integer species count

## How to apply

First retrieve the reported Venn-diagram counts verbatim from the article: inflow (I-source) segments B-I 774, N-I 669, G-I 548 and pond (P-source) segments B-P 882, N-P 553, G-P 415. Second, compute per-location totals (farms B, N, G) by combining the corresponding I- and P-source counts while correcting for shared/overlap segments shown in the diagrams, then derive unique (single-location) and shared-species counts for each location pair. Third, assemble all values into a tidy table with columns for source type (inflow/pond), location, and integer species count, and write it to venn_species_occurrence_counts.csv. The method is entirely manual expert review — no OTU re-clustering is performed — because the article delimited 1318 bacterial species via the SILVAngs pipeline, and only the diagram segments, not the underlying OTU table, are available.

## Examples

```
python -c "import pandas as pd; pd.DataFrame({'source':['inflow','inflow','inflow','pond','pond','pond'],'location':['B','N','G','B','N','G'],'species_count':[774,669,548,882,553,415]}).to_csv('venn_species_occurrence_counts.csv',index=False)"
```

## Evaluation signals

- Reconstructed I-source segment counts (B-I 774 + N-I 669 + G-I 548, minus overlap corrections) reconcile to the reported total of 1074 species
- Reconstructed P-source segment counts (B-P 882 + N-P 553 + G-P 415, minus overlap corrections) reconcile to the reported total of 1006 species
- Per-location sums (B, N, G) are consistent with the reported Venn-diagram segments after overlap correction
- Output CSV contains exactly the tidy schema: source type (inflow/pond), location, integer species count; all counts are non-negative integers
- Derived unique (single-location) and shared-species counts for each location pair sum, with overlaps, back to the per-source totals

## Limitations

- Exact farm locations were withheld at farmers' request, so per-location results cannot be tied to geographic coordinates or validated against external environmental data
- Only diagram segments and in-text counts are available; overlap segments must be inferred from the figures, so overlap correction is a manual, error-prone step
- The supplementary table 1 referenced for sequencing output/quality stats could not be retrieved (supplementary content was empty), preventing independent verification of the source counts

## Evidence

- [full_text] Total species counts across sources: [evidence span withheld — anchor status: not_found]
- [discussion] Farm locations withheld: "The exact location of these farms cannot be disclosed based on the farmer demand."
- [supplementary] Supplementary data unavailable: "The sequencing output and quality stats for each sample are presented in the supplementary table 1."
- [discussion] Pipeline that produced the delimited species: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline"
