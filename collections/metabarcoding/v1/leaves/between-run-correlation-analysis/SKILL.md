---
name: between-run-correlation-analysis
description: Use when assessing how a change in the OTU reference set (e.g. UCLUST-derived
  ab initio OTUs vs. the default hOTU reference) affects reproducibility of MAPseq
  OTU mapping across sequencing runs.
when_to_use_negative:
- Input is already a feature/OTU count table — this skill starts from raw reads and
  builds/replaces the reference OTU set before mapping
- The analysis goal is taxonomic classification accuracy rather than between-run consistency
  of OTU mapping
- Only a single sequencing run is available, so no pairwise between-run correlations
  or Jaccard overlaps can be computed
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAPseq
  role: Maps reads to the custom UCLUST OTU reference and the default hOTU reference,
    producing .mseq files and multi-sample OTU count tables via -otutable
  repo: https://github.com/jfmrod/mapseq
- name: UCLUST
  role: Builds the ab initio reference OTU set from reference rRNA sequences at the
    paper's OTU identity thresholds, used to replace the default hOTU reference
- name: R
  role: Imports the MAPseq OTU table and computes pairwise between-run abundance correlations
    (Spearman/Pearson) and Jaccard overlaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:84de0e0e2134a34e2d79c3cdb691dae9498e9e892c83e5d5147eaefbf083a448
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - R
  - UCLUST
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# between-run-correlation-analysis

## Summary

Quantifies between-run consistency of OTU mapping by replacing MAPseq's default pre-clustered hOTU reference with ab initio UCLUST-derived reference OTUs, then comparing pairwise abundance correlations (median 0.62) and Jaccard overlaps (median 0.33) of detected OTUs across runs against the default reference.

## When to use

Use when assessing how a change in the OTU reference set (e.g. UCLUST-derived ab initio OTUs vs. the default hOTU reference) affects reproducibility of MAPseq OTU mapping across sequencing runs.

## When NOT to use

- Input is already a feature/OTU count table — this skill starts from raw reads and builds/replaces the reference OTU set before mapping
- The analysis goal is taxonomic classification accuracy rather than between-run consistency of OTU mapping
- Only a single sequencing run is available, so no pairwise between-run correlations or Jaccard overlaps can be computed

## Inputs

- Reference rRNA sequence FASTA file (full-length ribosomal RNA sequences) for ab initio UCLUST clustering
- Demultiplexed, quality-filtered FASTA reads per sequencing run (e.g. rawseqs.fa)
- MAPseq .mseq classification outputs per run (produced with the custom UCLUST OTU reference and the default hOTU reference)

## Outputs

- Multi-sample OTU count table (TSV) per reference set, importable into R
- Pairwise between-run Spearman/Pearson abundance correlation values per reference set
- Pairwise Jaccard overlap values of detected OTUs per reference set
- Side-by-side median comparison table uclust_vs_hotu_comparison.csv (abundance-correlation and Jaccard-overlap columns for both reference sets)

## How to apply

First build an ab initio reference OTU set from the reference rRNA sequences with UCLUST at the paper's OTU identity thresholds, producing a FASTA and a tab-separated taxonomy file in MAPseq's format (header lines '#cutoff:', '#name:', '#levels:' followed by accession and taxonomy columns). Run MAPseq per run with this custom reference in place of the default hOTU reference (mapseq rawseqs.fa customref.fasta customref.tax > run.mseq), keeping the workflow independent of downstream mapping software. Generate the multi-sample OTU count table with 'mapseq -otutable' (a TSV with counts per sample column-wise and OTU labels row-wise) and import it into R via read.table(sep='	', header=TRUE). Compute all pairwise between-run Spearman/Pearson abundance correlations for the UCLUST-OTU profiles and the pairwise Jaccard overlap of detected OTUs between runs, then compare medians against the default hOTU reference profiles; the reported UCLUST-based medians are 0.62 (abundance correlation) and 0.33 (Jaccard overlap, Supplementary Fig. S3). Assemble a side-by-side median comparison table and write it to uclust_vs_hotu_comparison.csv.

## Related tools

- **MAPseq** (Maps reads to the custom UCLUST OTU reference and the default hOTU reference, producing .mseq files and multi-sample OTU count tables via -otutable) — https://github.com/jfmrod/mapseq
- **UCLUST** (Builds the ab initio reference OTU set from reference rRNA sequences at the paper's OTU identity thresholds, used to replace the default hOTU reference)
- **R** (Imports the MAPseq OTU table and computes pairwise between-run abundance correlations (Spearman/Pearson) and Jaccard overlaps)

## Examples

```
mapseq rawseqs.fa uclust_otus.fa uclust_otus.tax > rawseqs.fa.mseq && mapseq -otutable run1.mseq run2.mseq run3.mseq > uclust.otutable
```

## Evaluation signals

- The comparison table uclust_vs_hotu_comparison.csv contains both the abundance-correlation and Jaccard-overlap columns for both reference sets
- UCLUST-based medians reproduce the reported values within a reasonable tolerance: abundance correlation median 0.62 and Jaccard overlap median 0.33 (Supplementary Fig. S3)
- MAPseq runs with the custom reference succeed using a taxonomy file with the required '#cutoff:', '#name:', '#levels:' header and two tab-separated columns (accession id and taxonomy)
- The '-otutable' output is a TSV with counts per sample column-wise and OTU labels row-wise that imports cleanly into R with read.table(sep='	', header=TRUE)
- Correlations and Jaccard overlaps are computed over all pairwise run combinations for both the UCLUST-OTU and default hOTU reference profiles

## Limitations

- Lower between-run consistency (median correlation 0.62, Jaccard 0.33) with UCLUST-derived references is a property of the reference set, independent of downstream mapping software; results depend on the identity thresholds chosen for UCLUST clustering
- MAPseq's '-otutable' had a bug in versions before 1.2.6 that used only counts from the first sample, so version 1.2.6 or later is required for multi-sample OTU tables
- MAPseq's combined confidence is computed from a score confidence and an identity cutoff confidence; the README recommends a combined confidence cutoff of 0.4–0.5, and custom references must supply preoptimized identity cutoffs in the taxonomy header

## Evidence

- [readme] README (usage of -otutable): [evidence span withheld — anchor status: external_doc]
- [readme] README (custom reference usage): [evidence span withheld — anchor status: external_doc]
- [abstract] Article abstract (OTU mapping scope): "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies"
- [readme] README (recommended confidence cutoff): [evidence span withheld — anchor status: external_doc]
