---
name: reference-set-sensitivity-comparison
description: Use when you need to test whether OTU mapping results depend on the choice
  of reference OTU set — e.g., replacing MAPseq's default hOTU reference with UCLUST-clustered
  reference OTUs built at the paper's identity thresholds.
when_to_use_negative:
- Input is already an OTU abundance feature table — this skill requires re-running
  the mapping step against alternative references from raw demultiplexed reads
- The analysis goal is taxonomic classification accuracy (F½ score) rather than between-run
  consistency of OTU mapping across reference sets
- Only a single run/sample is available — between-run Spearman/Pearson correlations
  and Jaccard overlaps require multiple runs to pair
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MAPseq
  role: Reference-based OTU mapping of rRNA reads against both the default hOTU reference
    and the custom UCLUST-derived reference; -otutable produces the multi-sample OTU
    count tables used for the consistency metrics
  repo: https://github.com/jfmrod/mapseq
- name: UCLUST
  role: Generates the ab initio reference OTU clusters from the reference rRNA sequences
    at the paper's OTU identity thresholds, to replace the default hOTU reference
- name: R
  role: Imports the mapseq OTU table (read.table with sep="\t", header=TRUE) and computes
    pairwise between-run abundance correlations and Jaccard overlaps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:eabfd2770e5f2efc99891150a38df894211c8257f5d770b0672028436e2aadcd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - R
  - UCLUST
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reference-set-sensitivity-comparison

## Summary

A benchmarking skill that quantifies how swapping the OTU reference set (e.g., the default pre-clustered hOTU reference vs. ab initio UCLUST-clustered reference OTUs) changes between-run consistency of MAPseq OTU mapping, measured as pairwise between-run Spearman/Pearson abundance correlations and Jaccard overlaps of detected OTUs.

## When to use

Use when you need to test whether OTU mapping results depend on the choice of reference OTU set — e.g., replacing MAPseq's default hOTU reference with UCLUST-clustered reference OTUs built at the paper's identity thresholds.

## When NOT to use

- Input is already an OTU abundance feature table — this skill requires re-running the mapping step against alternative references from raw demultiplexed reads
- The analysis goal is taxonomic classification accuracy (F½ score) rather than between-run consistency of OTU mapping across reference sets
- Only a single run/sample is available — between-run Spearman/Pearson correlations and Jaccard overlaps require multiple runs to pair

## Inputs

- Reference rRNA sequence FASTA (e.g., full-length 16S reference sequences used to build OTUs)
- MAPseq default hOTU reference (shipped with the binary; data files must stay relative to the binary path)
- Demultiplexed, quality-filtered amplicon FASTA reads for multiple runs (e.g., rawseqs.fa per run)
- Custom reference taxonomy file(s) (two tab-separated columns: accession id and taxonomy, with a # header giving cutoffs, e.g., '#cutoff: 0.00:0.08 0.70:0.35 ...')

## Outputs

- Per-run MAPseq .mseq classification files (simple or confidences format via --outfmt)
- Per-run OTU/taxa count tables from mapseq -otutable (tab-separated, samples as columns, OTU labels as rows)
- Pairwise between-run Spearman/Pearson abundance correlation estimates per reference set
- Pairwise between-run Jaccard overlap of detected OTUs per reference set
- Side-by-side median comparison table written to uclust_vs_hotu_comparison.csv

## How to apply

Build an ab initio reference OTU set from the reference rRNA sequences with UCLUST at the paper's OTU identity thresholds, then run the MAPseq workflow with mapseq <reads.fa> <uclust_ref.fasta> <uclust_ref.tax> so reads are mapped against the custom reference instead of the default hOTU set, producing per-run .mseq outputs independent of downstream mapping software. Aggregate per-sample counts into OTU tables using mapseq -otutable over the multiple .mseq files. For both the UCLUST-based and the default hOTU-based profiles, compute all pairwise between-run Spearman/Pearson abundance correlations (reported median for UCLUST reference = 0.62) and Jaccard overlaps of detected OTUs between runs (reported median for UCLUST reference = 0.33, Supplementary Fig. S3). Assemble a side-by-side median comparison table for both reference sets and validate that the UCLUST-based medians reproduce the reported values within a reasonable tolerance.

## Related tools

- **MAPseq** (Reference-based OTU mapping of rRNA reads against both the default hOTU reference and the custom UCLUST-derived reference; -otutable produces the multi-sample OTU count tables used for the consistency metrics) — https://github.com/jfmrod/mapseq
- **UCLUST** (Generates the ab initio reference OTU clusters from the reference rRNA sequences at the paper's OTU identity thresholds, to replace the default hOTU reference)
- **R** (Imports the mapseq OTU table (read.table with sep="\t", header=TRUE) and computes pairwise between-run abundance correlations and Jaccard overlaps)

## Examples

```
mapseq rawseqs.fa uclust_ref.fasta uclust_ref.tax > rawseqs.fa.mseq && mapseq -otutable run1.mseq run2.mseq run3.mseq > map.otutable
```

## Evaluation signals

- The uclust_vs_hotu_comparison.csv contains both abundance-correlation and Jaccard-overlap columns for both reference sets (hOTU and UCLUST)
- UCLUST-reference medians reproduce the reported values — median abundance correlation 0.62 and median Jaccard overlap 0.33 (Supplementary Fig. S3) — within a reasonable tolerance
- Per-run .mseq and OTU-table outputs were generated independently of any downstream mapping software, using mapseq against each reference set
- The custom reference run used the '#cutoff'-headered taxonomy file format MAPseq expects (accession id + taxonomy per reference sequence)
- The result direction is stable across mapping software: UCLUST-reference medians are lower than default-hOTU medians, independent of the downstream mapping software

## Limitations

- The default reference must be installed relative to the MAPseq binary path — moving only the binary breaks the bundled reference data files
- MAPseq recommends a combined confidence cutoff of 0.4–0.5 (highest F½-score in benchmarks); the default v1.2.6 -otutable option had a bug using only counts from the first sample, fixed in v1.2.6
- MAPseq's default OTU taxonomies come with precomputed hOTU clusters; custom references require supplying one or more taxonomy files, so the comparison depends on the quality and thresholds of the UCLUST clustering

## Evidence

- [readme] custom reference usage: [evidence span withheld — anchor status: external_doc]
- [readme] OTU count table output: [evidence span withheld — anchor status: external_doc]
- [abstract] hierarchical OTU mapping: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences"
- [readme] taxonomy file format: [evidence span withheld — anchor status: external_doc]
