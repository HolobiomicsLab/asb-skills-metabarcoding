---
name: species-occurrence-count-extraction
description: Use when a task asks to reproduce per-location, per-source species occurrence
  counts from a metabarcoding paper's Venn diagram (I-sources vs. P-sources) rather
  than re-pipeline raw reads.
when_to_use_negative:
- The goal is to recompute species richness de novo from raw FASTQ reads — use the
  metabarcoding pipeline (SILVAngs, BBmerge, BBduk) instead of extracting reported
  counts
- The source article lacks Venn-diagram or per-location segment counts, so no counts
  can be reconciled
- The analysis requires continuous abundance or relative-frequency data — this skill
  handles integer occurrence (presence) counts only
edam_operation: http://edamontology.org/operation_0306
tools:
- name: SILVAngs
  role: Online pipeline whose species delimitation produced the reported occurrence
    counts being extracted; not re-run here
- name: manual expert review
  role: Counts are read from the article's Venn diagrams and reconciled manually,
    not by automated parsing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:d3fd0371079febe265ac90b973ebc2478a1079d21f568e6ad8c11fd1003011ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SILVAngs
  - manual expert review
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# species-occurrence-count-extraction

## Summary

Extracts and reconciles bacterial species occurrence counts per water source (inflow vs. pond) and trout farm location (N, B, G) from published Venn-diagram numbers, assembling them into a tidy occurrence-count table. It reproduces reported source totals rather than re-quantifying reads.

## When to use

Use when a task asks to reproduce per-location, per-source species occurrence counts from a metabarcoding paper's Venn diagram (I-sources vs. P-sources) rather than re-pipeline raw reads.

## When NOT to use

- The goal is to recompute species richness de novo from raw FASTQ reads — use the metabarcoding pipeline (SILVAngs, BBmerge, BBduk) instead of extracting reported counts
- The source article lacks Venn-diagram or per-location segment counts, so no counts can be reconciled
- The analysis requires continuous abundance or relative-frequency data — this skill handles integer occurrence (presence) counts only

## Inputs

- Published Venn-diagram segment counts for I-sources (B-I, N-I, G-I) and P-sources (B-P, N-P, G-P)
- Reported source totals (1074 I-source species; 1006 P-source species)
- Source card specification listing expected counts and the target output filename

## Outputs

- venn_species_occurrence_counts.csv: tidy table with columns source type (inflow/pond), location (N, B, G), and integer species count
- Derived per-location totals and unique/shared species counts per location pair

## How to apply

Retrieve the reported Venn-diagram counts: I-sources total 1074 species (B-I 774, N-I 669, G-I 548) and P-sources total 1006 species (B-P 882, N-P 553, G-P 415). Compute per-location totals (B, N, G) by combining the corresponding I-source and P-source counts, correcting for overlap segments in the reported diagrams. Derive the unique (single-location) species counts and reported shared-species counts for each location pair from the Venn-diagram segments. Assemble a tidy table with columns for source type (inflow/pond), location, and integer species count, and write it to venn_species_occurrence_counts.csv. The counts originate from SILVAngs-based 16S rDNA analysis, so they reflect that pipeline's species delimitation (identity criteria 1.00 and 0.98, unclassified when (% sequence identity + % alignment coverage)/2 ≤ 93). Validate that reconstructed source totals reconcile exactly to 1074 (I-sources) and 1006 (P-sources).

## Related tools

- **SILVAngs** (Online pipeline whose species delimitation produced the reported occurrence counts being extracted; not re-run here)
- **manual expert review** (Counts are read from the article's Venn diagrams and reconciled manually, not by automated parsing)

## Examples

```
import pandas as pd; pd.DataFrame({'source':['inflow']*3+['pond']*3,'location':['B','N','G','B','N','G'],'species_count':[774,669,548,882,553,415]}).to_csv('venn_species_occurrence_counts.csv',index=False)
```

## Evaluation signals

- Reconstructed I-source total equals the reported 1074 species (774 + 669 + 548 with overlap corrections)
- Reconstructed P-source total equals the reported 1006 species (882 + 553 + 415 with overlap corrections)
- Per-location sums (B, N, G) are consistent with the reported Venn-diagram segments for both source types
- All values in venn_species_occurrence_counts.csv are non-negative integers with exactly one row per source-type × location combination
- Unique plus shared segments sum to the per-source total for each location

## Limitations

- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility of the counts
- Supplementary table 1 with sequencing output and quality stats is referenced but the supplementary content could not be retrieved, so counts cannot be cross-checked against per-sample data
- Counts depend on SILVAngs species delimitation thresholds; some OTUs remained unclassified when (% sequence identity + % alignment coverage)/2 did not exceed 93
- Overlap corrections rely on visually reading Venn-diagram segments, which can introduce transcription error

## Evidence

- [results] I-source total and per-location counts: [evidence span withheld — anchor status: not_found]
- [intro] Study design grounding the occurrence counts: "We aimed to profile both prokaryotic (16S) and eukaryotic (ITS) microbial community of three different semi-intensive freshwater trout farms"
- [discussion] Pipeline producing the species counts: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] Location disclosure limitation: "The exact location of these farms cannot be disclosed based on the farmer demand."
- [discussion] Missing supplementary data for cross-validation: "The sequencing output and quality stats for each sample are presented in the supplementary table 1."
