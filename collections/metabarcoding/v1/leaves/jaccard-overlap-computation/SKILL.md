---
name: jaccard-overlap-computation
description: Use when you have per-run OTU abundance tables or OTU call lists (e.g.,
  MAPseq 'map.otutable' TSVs from multiple .mseq files) and need to quantify between-run
  reproducibility of OTU identifications via Jaccard overlap.
when_to_use_negative:
- Input is already a merged, pre-collapsed feature/abundance table where OTU detection
  calls have been altered post-mapping — recompute from raw per-run MAPseq outputs
  instead
- The question is about taxonomic composition differences rather than between-run
  consistency of OTU identifications (use abundance correlations or differential abundance
  tests instead)
- Runs were mapped against different reference OTU sets, making OTU labels non-comparable
  without harmonization
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MAPseq
  role: Produces per-run OTU assignments and the multi-sample OTU count table ('-otutable')
    whose detected-OTU columns are compared via Jaccard overlap
  repo: https://github.com/jfmrod/mapseq
- name: UCLUST
  role: Generates the ab initio reference OTU set clustered at the paper's OTU identity
    thresholds, used as the alternative mapping reference in the Jaccard comparison
- name: R
  role: Imports the map.otutable TSV and computes pairwise Jaccard overlaps and medians
    between runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:971b463fa8aa33970b82417ec8d1d9a537215a7634e3644a3bdc614b069b9bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - R
  - UCLUST
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# jaccard-overlap-computation

## Summary

Compute pairwise Jaccard overlap of detected OTUs between sequencing/mapping runs to quantify between-run consistency of OTU identifications, as used to benchmark MAPseq OTU mappings against alternative reference OTU sets (e.g., default hOTU vs. UCLUST-derived references, reported median Jaccard 0.33 for UCLUST-based mapping).

## When to use

Use when you have per-run OTU abundance tables or OTU call lists (e.g., MAPseq 'map.otutable' TSVs from multiple .mseq files) and need to quantify between-run reproducibility of OTU identifications via Jaccard overlap.

## When NOT to use

- Input is already a merged, pre-collapsed feature/abundance table where OTU detection calls have been altered post-mapping — recompute from raw per-run MAPseq outputs instead
- The question is about taxonomic composition differences rather than between-run consistency of OTU identifications (use abundance correlations or differential abundance tests instead)
- Runs were mapped against different reference OTU sets, making OTU labels non-comparable without harmonization

## Inputs

- Per-sample MAPseq .mseq classification output files
- MAPseq OTU count table (map.otutable TSV) with one column per sample/run and rows as OTU labels
- Reference OTU sets (default hOTU reference and alternative e.g. UCLUST-clustered reference fasta + taxonomy files)

## Outputs

- Pairwise between-run Jaccard overlap matrix (runs × runs)
- Summary statistic (median Jaccard overlap) per reference OTU set
- Comparison table (e.g., uclust_vs_hotu_comparison.csv) of medians for the two reference sets

## How to apply

Run MAPseq OTU mapping per run to produce .mseq outputs, then generate a count table with 'mapseq -otutable <sample1.mseq> <sample2.mseq> ...', which emits a tab-separated file with counts per sample (columns) and OTU or taxonomic labels (rows); import it into R with read.table. Binarize each sample column to a detected/not-detected OTU vector, compute all pairwise Jaccard indices between runs (|A∩B|/|A∪B| over detected OTU sets), and summarize the distribution (e.g., median). For reference-set comparisons, repeat identically for both the default hOTU reference profiles and the alternative (e.g., UCLUST-clustered at the paper's OTU identity thresholds) profiles and assemble side-by-side medians; apply the same combined-confidence filtering across runs, using MAPseq's recommended combined confidence cutoff of 0.4–0.5 to define 'detected'.

## Related tools

- **MAPseq** (Produces per-run OTU assignments and the multi-sample OTU count table ('-otutable') whose detected-OTU columns are compared via Jaccard overlap) — https://github.com/jfmrod/mapseq
- **UCLUST** (Generates the ab initio reference OTU set clustered at the paper's OTU identity thresholds, used as the alternative mapping reference in the Jaccard comparison)
- **R** (Imports the map.otutable TSV and computes pairwise Jaccard overlaps and medians between runs)

## Examples

```
mapseq -otutable sample1.mseq sample2.mseq sample3.mseq > map.otutable; myotutable <- read.table("map.otutable",sep="\t",header=TRUE)
```

## Evaluation signals

- The OTU table contains one column per sample/run and row labels that are OTU identifiers, not just taxonomy strings
- Every pair of runs yields a Jaccard value in [0, 1]; identical runs must give 1.0, disjoint OTU sets 0.0
- The Jaccard matrix is symmetric with a diagonal of 1.0 and has N(N-1)/2 unique pairwise values for N runs
- Median Jaccard overlap for the UCLUST-based mapping reproduces the reported value 0.33 (Supplementary Fig. S3) within reasonable tolerance
- The comparison output table contains both abundance-correlation and Jaccard-overlap columns for both reference sets (hOTU and UCLUST)

## Limitations

- Jaccard overlap depends on the detection/confidence threshold: MAPseq's recommended combined confidence cutoff of 0.4 or 0.5 (highest F1/2-score in benchmarks) defines which OTUs count as 'detected', so changing it changes the overlaps
- OTU labels are reference-set specific — Jaccard values are only comparable when both runs use the same reference OTU definitions
- Jaccard ignores abundance and is binary, so it should be reported alongside between-run abundance correlations (e.g., median Spearman/Pearson 0.62 for the UCLUST reference) to capture composition differences

## Evidence

- [readme] Generates a tab separated value (tsv) file with the counts for each sample (column wise) and OTU or taxonomic labels (row wise).: [evidence span withheld — anchor status: external_doc]
- [abstract] providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies"
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] The generated table can be imported into R with the following R command: [evidence span withheld — anchor status: external_doc]
