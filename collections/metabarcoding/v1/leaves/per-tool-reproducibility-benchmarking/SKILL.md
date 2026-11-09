---
name: per-tool-reproducibility-benchmarking
description: Use when the same biological samples have been sequenced in two independent
  runs and you need to quantify how consistently competing rRNA classification tools
  assign OTUs and report correlated abundances across runs.
when_to_use_negative:
- Input is already a single feature/OTU count table with no paired-run replicate data
  — reproducibility between runs cannot be computed
- The question is absolute classification accuracy against a ground truth rather than
  consistency between technical/sequencing replicates
- Samples were not sequenced in two independent runs with different rRNA regions,
  so no cross-run overlap exists to compare
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MAPseq
  role: Primary tool under test; classifies reads against NCBI taxonomy and OTU taxonomies
    in a single run and produces .mseq outputs convertible to per-sample count tables
    via -otucounts/-otutable
  repo: https://github.com/jfmrod/mapseq
- name: VSEARCH
  role: Competing rRNA classification baseline run with equivalent reference settings
    for cross-tool reproducibility comparison
- name: NINJA-OPS
  role: Competing rRNA classification baseline run with equivalent reference settings
    for cross-tool reproducibility comparison
- name: USEARCH
  role: Competing rRNA classification baseline run with equivalent reference settings
    for cross-tool reproducibility comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:6d26d262c5be9f64cf92e1350b20490eb627baa0298124fcbd97fb6a947a3440
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - NINJA-OPS
  - USEARCH
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# per-tool-reproducibility-benchmarking

## Summary

A benchmarking skill for quantifying cross-run reproducibility of rRNA read classification/OTU mapping tools: run each tool (MAPseq, VSEARCH, NINJA-OPS, USEARCH) on both independent sequencing runs of the same samples, then compare per-tool Spearman correlations of OTU abundances and median per-sample Jaccard overlap of shared OTU identifications.

## When to use

Use when the same biological samples have been sequenced in two independent runs and you need to quantify how consistently competing rRNA classification tools assign OTUs and report correlated abundances across runs.

## When NOT to use

- Input is already a single feature/OTU count table with no paired-run replicate data — reproducibility between runs cannot be computed
- The question is absolute classification accuracy against a ground truth rather than consistency between technical/sequencing replicates
- Samples were not sequenced in two independent runs with different rRNA regions, so no cross-run overlap exists to compare

## Inputs

- FASTA/FASTQ files of demultiplexed, quality-filtered rRNA amplicon reads from each sample in both sequencing runs
- Reference full-length rRNA sequence sets with taxonomies and precomputed OTU clusters (e.g. MAPseq's default NCBI + OTU reference)
- List of competing tools to benchmark with their equivalent reference settings (VSEARCH, NINJA-OPS, USEARCH)

## Outputs

- Per-sample .mseq classification files per tool and per run
- Spearman correlation of OTU abundances between run 1 and run 2 for each tool
- Median per-sample Jaccard index of shared OTU identifications for each tool
- Consolidated reproducibility metrics table (hmp_reproducibility_metrics.csv) ordered by tool

## How to apply

Load the paired-run amplicon dataset (e.g. 2194 Human Microbinome Project samples sequenced in two independent runs over different rRNA gene regions) and run MAPseq 1.2.6 on every sample from both runs to obtain taxonomy classifications and hierarchical OTU mappings, then run each baseline tool (VSEARCH, NINJA-OPS, USEARCH) with equivalent reference settings. For each tool, restrict to OTUs shared between the two runs and compute the Spearman correlation of OTU abundances across all samples between run 1 and run 2. Then compute the per-sample Jaccard index of shared OTU identifications between the two runs and record the median per tool. Assemble per-tool correlation values and median Jaccard values into a single ordered table (e.g. hmp_reproducibility_metrics.csv) and validate that all Jaccard values lie in [0, 1] and that the expected best tool ranks first. MAPseq's recommended combined confidence cutoff of 0.4–0.5 should be applied when generating its classifications, as this yielded the highest F1/2-score in the authors' benchmarks.

## Related tools

- **MAPseq** (Primary tool under test; classifies reads against NCBI taxonomy and OTU taxonomies in a single run and produces .mseq outputs convertible to per-sample count tables via -otucounts/-otutable) — https://github.com/jfmrod/mapseq
- **VSEARCH** (Competing rRNA classification baseline run with equivalent reference settings for cross-tool reproducibility comparison)
- **NINJA-OPS** (Competing rRNA classification baseline run with equivalent reference settings for cross-tool reproducibility comparison)
- **USEARCH** (Competing rRNA classification baseline run with equivalent reference settings for cross-tool reproducibility comparison)

## Examples

```
mapseq -nthreads 8 rawseqs.fa > rawseqs.fa.mseq; mapseq -otutable run1_sample1.mseq run2_sample1.mseq run1_sample2.mseq run2_sample2.mseq > map.otutable
```

## Evaluation signals

- All computed per-sample Jaccard indices fall within [0, 1]
- Reproduced median Jaccard for MAPseq equals 0.52, exceeding VSEARCH, NINJA-OPS, and USEARCH (Fig. 1g-h)
- Spearman abundance correlations are computed only over OTUs shared between the two runs and across all 2194 samples
- hmp_reproducibility_metrics.csv contains one row per tool with both a correlation value and a median Jaccard value, ordered by tool
- MAPseq classifications used a combined confidence cutoff of 0.4–0.5, consistent with the benchmark setting

## Limitations

- MAPseq confidence estimates depend on reference coverage: the identity cutoff confidence guards against misclassification when no representative of the true classification exists in the database
- Older MAPseq versions had bugs affecting OTU tables — fixed in 1.2.5 ('-otutable' option) and 1.2.6 ('-otutable' using only counts from first sample), so version 1.2.6 should be used
- Cross-run Jaccard overlap is bounded by the differing rRNA gene regions sequenced in the two runs, so median overlaps are moderate (best tool median = 0.52) rather than near 1

## Evidence

- [abstract] Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] Generates a tab separated value (tsv) file with the counts for each sample (column wise) and OTU or taxonomic labels (row wise).: [evidence span withheld — anchor status: external_doc]
- [readme] by default mapseq maps reads to the NCBI taxonomy and to OTU taxonomies: [evidence span withheld — anchor status: external_doc]
