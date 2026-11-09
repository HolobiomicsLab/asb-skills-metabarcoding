---
name: otu-reference-construction-with-uclust
description: Use when you need to test how the choice of reference OTU database (e.g.
  UCLUST-derived ab initio clusters vs. the default hOTU reference) affects MAPseq
  OTU mapping consistency between runs.
when_to_use_negative:
- The analysis goal is taxonomy classification only, not hierarchical OTU mapping
  — MAPseq provides both, but this skill specifically evaluates OTU reference consistency
- Input is already a count/feature table; this skill operates upstream on raw demultiplexed
  reads and reference sequence sets
- You need the maximum between-run consistency rather than an ab initio reference
  — the default hOTU reference showed higher medians (correlation and Jaccard) than
  the UCLUST-derived one
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: UCLUST
  role: Clusters the reference rRNA sequences into ab initio reference OTUs at the
    paper's OTU identity thresholds
- name: MAPseq
  role: Maps reads against the UCLUST-generated reference OTU set in place of the
    default hOTU reference, producing per-run .mseq outputs and OTU count tables
  repo: https://github.com/jfmrod/mapseq
- name: R
  role: Imports the MAPseq OTU table and computes pairwise between-run Spearman/Pearson
    correlations and Jaccard overlaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:bc4c5d5f1c7457954ddc427bebf5ca3a42edda7f6a42dbc1cfd06ed8b5a367b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - R
  - UCLUST
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# otu-reference-construction-with-uclust

## Summary

Builds an ab initio reference OTU set from reference rRNA sequences using UCLUST at the paper's OTU identity thresholds, then swaps it into the MAPseq OTU mapping workflow in place of the default pre-clustered hOTU reference to quantify how the reference choice affects between-run consistency of OTU mapping.

## When to use

Use when you need to test how the choice of reference OTU database (e.g. UCLUST-derived ab initio clusters vs. the default hOTU reference) affects MAPseq OTU mapping consistency between runs.

## When NOT to use

- The analysis goal is taxonomy classification only, not hierarchical OTU mapping — MAPseq provides both, but this skill specifically evaluates OTU reference consistency
- Input is already a count/feature table; this skill operates upstream on raw demultiplexed reads and reference sequence sets
- You need the maximum between-run consistency rather than an ab initio reference — the default hOTU reference showed higher medians (correlation and Jaccard) than the UCLUST-derived one

## Inputs

- Reference full-length rRNA sequences (nucleotide FASTA) from which OTUs are clustered
- Custom reference taxonomy file (tab-separated accession-to-OTU assignment with a # header declaring cutoffs and levels)
- Demultiplexed, quality-filtered amplicon sequence reads (FASTA/FASTQ) per run to be mapped
- Default MAPseq hOTU reference profiles for the between-run comparison baseline

## Outputs

- UCLUST-generated ab initio reference OTU set (clustered at the paper's OTU identity thresholds)
- Per-run abundance tables for the UCLUST-OTU mapping, produced via MAPseq .mseq output and 'mapseq -otutable'
- Pairwise between-run Spearman/Pearson abundance correlations and Jaccard overlap values (reported medians: 0.62 and 0.33 for the UCLUST reference)
- Side-by-side comparison table of medians for UCLUST vs. hOTU reference sets (uclust_vs_hotu_comparison.csv)

## How to apply

First cluster the reference full-length rRNA sequences into OTUs with UCLUST at the paper's OTU identity thresholds, producing an ab initio reference OTU set. Then run the MAPseq OTU mapping workflow with this UCLUST-generated reference in place of the default hOTU reference, supplying the custom fasta reference and taxonomy files to mapseq so per-run abundance tables are produced independent of any downstream mapping software. MAPseq accepts user references as 'mapseq rawseqs.fa <customref.fasta> <customref.tax>' and can summarize multiple runs with 'mapseq -otutable sample1.mseq sample2.mseq ...'. Compute pairwise between-run Spearman/Pearson abundance correlations for the UCLUST-OTU profiles and pairwise Jaccard overlap of detected OTUs between runs, then compare against the default hOTU reference profiles. The rationale is to isolate the effect of reference construction on reproducibility: in the source study the UCLUST-derived reference yielded lower between-run abundance correlations (median = 0.62) and lower Jaccard overlaps (median = 0.33) than the default reference, independent of the downstream mapping software. Assemble a side-by-side medians comparison table (e.g. uclust_vs_hotu_comparison.csv) with abundance-correlation and Jaccard-overlap columns for both reference sets.

## Related tools

- **UCLUST** (Clusters the reference rRNA sequences into ab initio reference OTUs at the paper's OTU identity thresholds)
- **MAPseq** (Maps reads against the UCLUST-generated reference OTU set in place of the default hOTU reference, producing per-run .mseq outputs and OTU count tables) — https://github.com/jfmrod/mapseq
- **R** (Imports the MAPseq OTU table and computes pairwise between-run Spearman/Pearson correlations and Jaccard overlaps)

## Examples

```
mapseq rawseqs.fa uclust_ref.fasta uclust_ref.tax > rawseqs.fa.mseq && mapseq -otutable run1.mseq run2.mseq run3.mseq > map.otutable
```

## Evaluation signals

- UCLUST-based median between-run abundance correlation reproduces the reported value of 0.62 within a reasonable tolerance
- UCLUST-based median Jaccard overlap reproduces the reported value of 0.33, matching Supplementary Fig. S3
- The comparison table uclust_vs_hotu_comparison.csv contains both abundance-correlation and Jaccard-overlap columns for both reference sets
- MAPseq OTU tables are generated from the .mseq outputs via 'mapseq -otutable' independently of any downstream mapping software
- Custom reference taxonomy file includes the required '#cutoff:' header line declaring identity cutoffs so MAPseq applies the intended OTU levels

## Limitations

- Using UCLUST-derived reference OTUs resulted in lower between-run consistency (median abundance correlation 0.62, median Jaccard overlap 0.33) than the default pre-clustered hOTU reference, independent of downstream mapping software
- MAPseq data files are searched for relative to the binary's path, so moving only the binary breaks the installation
- MAPseq expects reads to be previously demultiplexed and quality filtered before classification
- The custom taxonomy file format requires a # header with identity cutoff parameters; omitting it or mismatching cutoffs changes the OTU level definitions

## Evidence

- [other] Using reference OTUs clustered with UCLUST resulted in lower between-run abundance correlations (median = 0.62) and Jaccard overlaps (median = 0.33) than the default reference, independent of the downstream mapping software.: [evidence span withheld — anchor status: not_found]
- [readme] You can use mapseq with your own fasta reference and taxonomy files with the following command: [evidence span withheld — anchor status: external_doc]
- [readme] The taxonomy file should have a header (preceeded with the # character) with the identity cutoff parameters and description of the taxonomy followed by two tab-separated columns composed of the accession id and the taxonomy.: [evidence span withheld — anchor status: external_doc]
- [readme] Generates a tab separated value (tsv) file with the counts for each sample (column wise) and OTU or taxonomic labels (row wise).: [evidence span withheld — anchor status: external_doc]
- [readme] MAPseq takes as input a fasta file with raw sequence data which should have been previously demultiplexed and quality filtered: [evidence span withheld — anchor status: external_doc]
