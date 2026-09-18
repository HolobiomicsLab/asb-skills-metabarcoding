---
name: rrna-abundance-profiling-across-runs
description: Use when comparing OTU mapping reproducibility across sequencing runs,
  e.g. testing how swapping the default hOTU reference for UCLUST-clustered reference
  OTUs changes abundance correlations and shared OTU identifications.
when_to_use_negative:
- You only need taxonomy assignment, not OTU-level abundance profiling — plain `mapseq
  rawseqs.fa > rawseqs.fa.mseq` classification suffices
- Input is already a feature/OTU count table; re-running the MAPseq mapping would
  be redundant
- You intend to change downstream mapping software as the variable of interest — the
  finding shows UCLUST-reference effects hold 'independent of the downstream mapping
  software', so this comparison is not a software benchmark
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: MAPseq
  role: Reference-based rRNA classifier used to map each run's reads to the UCLUST-generated
    and default hOTU reference sets and produce per-run OTU abundance tables via -otutable
  repo: https://github.com/jfmrod/mapseq
- name: UCLUST
  role: Clusters the reference rRNA sequences at the paper's OTU identity thresholds
    to build the ab initio reference OTU set
- name: R
  role: Imports the map.otutable TSV and computes pairwise Spearman/Pearson abundance
    correlations and Jaccard overlaps between runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:7029a4f14075c029bf89dbfed28d5248a6a7a6dabc0e5be9dfeb20511677aca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - R
  - UCLUST
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rrna-abundance-profiling-across-runs

## Summary

Build per-run rRNA OTU abundance profiles with MAPseq under different reference OTU definitions (default pre-clustered hOTU vs. ab initio UCLUST-derived reference), then quantify between-run consistency via pairwise Spearman/Pearson abundance correlations and Jaccard overlap of detected OTUs.

## When to use

Use when comparing OTU mapping reproducibility across sequencing runs, e.g. testing how swapping the default hOTU reference for UCLUST-clustered reference OTUs changes abundance correlations and shared OTU identifications.

## When NOT to use

- You only need taxonomy assignment, not OTU-level abundance profiling — plain `mapseq rawseqs.fa > rawseqs.fa.mseq` classification suffices
- Input is already a feature/OTU count table; re-running the MAPseq mapping would be redundant
- You intend to change downstream mapping software as the variable of interest — the finding shows UCLUST-reference effects hold 'independent of the downstream mapping software', so this comparison is not a software benchmark

## Inputs

- Reference rRNA nucleotide sequences (FASTA) for building the ab initio UCLUST reference OTU set
- Demultiplexed, quality-filtered raw sequence FASTA files per sequencing run (e.g. rawseqs.fa)
- Custom reference taxonomy file(s) with identity-cutoff header (e.g. #cutoff: lines) for user-provided references
- Default MAPseq hOTU reference (bundled with MAPseq, e.g. mapref-2.2b) for the baseline profiles

## Outputs

- Per-run .mseq MAPseq classification files
- Multi-sample OTU count table (map.otutable, TSV, samples as columns, OTUs/taxa as rows)
- Pairwise between-run abundance correlation values (Spearman and Pearson) for each reference set
- Pairwise Jaccard overlap matrices of detected OTUs per reference set
- uclust_vs_hotu_comparison.csv: side-by-side median comparison table (UCLUST median correlation 0.62, Jaccard 0.33 vs. default hOTU reference)

## How to apply

First build an ab initio reference OTU set by clustering the reference rRNA sequences with UCLUST at the paper's OTU identity thresholds. Run MAPseq's OTU mapping workflow (mapseq on demultiplexed, quality-filtered FASTA input, using the custom reference via `mapseq rawseqs.fa <customref.fasta> <customref.tax>`) once per run with the UCLUST reference in place of the default hOTU reference, generating .mseq outputs independent of any downstream mapping software; summarize counts with `mapseq -otutable` across runs. In R, load the resulting map.otutable (read.table with sep='\t', header=TRUE) and compute all pairwise between-run Spearman/Pearson abundance correlations of the UCLUST-OTU profiles. Separately compute pairwise Jaccard overlap of detected OTUs between runs. Finally, assemble a side-by-side comparison table of medians for the UCLUST-based and default hOTU-based mappings and write it to uclust_vs_hotu_comparison.csv.

## Related tools

- **MAPseq** (Reference-based rRNA classifier used to map each run's reads to the UCLUST-generated and default hOTU reference sets and produce per-run OTU abundance tables via -otutable) — https://github.com/jfmrod/mapseq
- **UCLUST** (Clusters the reference rRNA sequences at the paper's OTU identity thresholds to build the ab initio reference OTU set)
- **R** (Imports the map.otutable TSV and computes pairwise Spearman/Pearson abundance correlations and Jaccard overlaps between runs)

## Examples

```
mapseq run1.fa ref_uclust.fa ref_uclust.tax > run1.fa.mseq; mapseq -otutable run1.fa.mseq run2.fa.mseq run3.fa.mseq > map.otutable
```

## Evaluation signals

- The UCLUST-based between-run abundance-correlation median reproduces the reported 0.62 within reasonable tolerance of the Supplementary Fig. S3 values
- The UCLUST-based Jaccard-overlap median reproduces the reported 0.33 within reasonable tolerance of Supplementary Fig. S3
- uclust_vs_hotu_comparison.csv contains both abundance-correlation and Jaccard-overlap columns for both reference sets (UCLUST-derived and default hOTU)
- map.otutable imports cleanly in R via read.table(sep='\t', header=TRUE) and includes counts for all runs (note: v1.2.6 fixed '-otutable' using only counts from first sample)
- Per-run profiles were produced with the same mapping software throughout, so reference choice — not downstream software — is the only variable

## Limitations

- Using UCLUST-derived reference OTUs lowers between-run consistency (median correlation 0.62, Jaccard 0.33) relative to the default pre-clustered hOTU reference, independent of downstream mapping software
- MAPseq searches for data files relative to the binary's path — moving only the binary breaks the installation
- A combined confidence cutoff of 0.4 or 0.5 is recommended (highest F1/2-score in benchmarks); the value chosen affects assignments
- Older MAPseq versions had '-otutable' bugs (only first-sample counts, fixed in v1.2.6) and double hits for long queries >1200 bp (fixed in v1.2.3)

## Evidence

- [readme] You can use mapseq with your own fasta reference and taxonomy files with the following command:: [evidence span withheld — anchor status: external_doc]
- [readme] Generates a tab separated value (tsv) file with the counts for each sample (column wise) and OTU or taxonomic labels (row wise).: [evidence span withheld — anchor status: external_doc]
- [readme] myotutable <- read.table("map.otutable",sep="\t",header=TRUE): [evidence span withheld — anchor status: external_doc]
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] MAPseq is a set of fast and accurate sequence read classification tools designed to assign taxonomy and OTU classifications to ribosomal RNA sequences.: [evidence span withheld — anchor status: external_doc]
