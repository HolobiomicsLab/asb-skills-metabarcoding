---
name: 16s-mock-community-benchmarking
description: Use when evaluating whether a DADA2-based amplicon workflow recovers
  expected taxon counts, strain-level micro-diversity, and abundance patterns from
  mock communities, or when measuring its wall-clock runtime scaling (1 vs 4 cores)
  on small datasets.
when_to_use_negative:
- The input is already a feature/ASV table or processed OTU table rather than raw
  fastq reads
- You need relative read abundances to serve as estimates of true taxon abundances
  — the mock community results show read abundance does not correlate with ground-truth
  taxon abundance
- You need to resolve strain-level micro-diversity where classification confidence
  matters — 1 of the 3 Fusarium strains was misclassified despite correct micro-diversity
  detection
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_0092
tools:
- name: dadasnake
  role: Snakemake pipeline executing the full DADA2-based amplicon processing and
    mock community benchmarking runs
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Quality filtering, trimming, error estimation, denoising (ASV inference),
    and chimera removal
  repo: http://benjjneb.github.io/dada2/
- name: cutadapt
  role: Primer removal (and poly-G-tail removal) prior to DADA2 processing
- name: mothur
  role: Bayesian taxonomic classification of ASVs against UNITE/SILVA reference data
    (cutoff 60)
- name: BLAST
  role: Sequence similarity search whose tabular hits are summarized with BASTA for
    taxonomic hand-off
- name: seqtk
  role: Optional downsampling of reads per sample to control dataset size in benchmark
    runs
  repo: https://github.com/lh3/seqtk
merged_aliases:
- mock-community-asv-benchmarking
- mock-community-composition-benchmarking
merged_alias_records:
- alias: mock-community-asv-benchmarking
  slug: mock-community-asv-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: mock-community-composition-benchmarking
  slug: mock-community-composition-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:f11a4090f0ddda964f7587cb2b91724f55e33283ea36473aa135a518c9bf0d6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - DADA2
  - conda
  - cutadapt
  - dadasnake
  - mothur
  - seqtk
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# 16s-mock-community-benchmarking

## Summary

Benchmark an amplicon ASV pipeline (dadasnake/DADA2) against ground-truth mock communities of bacteria/archaea (16S, e.g. ERR777696, PRJNA517390) and fungi (ITS2, PRJNA626434) by measuring wall-clock runtime at different core counts, strain-resolution of intra-genus micro-diversity, and correspondence of relative read abundances to known mock compositions.

## When to use

Use when evaluating whether a DADA2-based amplicon workflow recovers expected taxon counts, strain-level micro-diversity, and abundance patterns from mock communities, or when measuring its wall-clock runtime scaling (1 vs 4 cores) on small datasets.

## When NOT to use

- The input is already a feature/ASV table or processed OTU table rather than raw fastq reads
- You need relative read abundances to serve as estimates of true taxon abundances — the mock community results show read abundance does not correlate with ground-truth taxon abundance
- You need to resolve strain-level micro-diversity where classification confidence matters — 1 of the 3 Fusarium strains was misclassified despite correct micro-diversity detection

## Inputs

- Raw paired-end fastq files of 16S V4 and/or ITS2 amplicon libraries (e.g. PRJNA517390 24-sample dataset with 2.9 million reads; fungal mock community PRJNA626434)
- Tab-separated sample table with 'library', 'r1_file', 'r2_file' columns
- dadasnake YAML configuration file (output_directory, raw_directory, sample_table path)
- Ground-truth mock community composition tables from Zenodo [60]
- Reference databases: SILVA (e.g. silva_nr99_v138_train_set.fa.gz) for 16S, UNITE v8 for ITS

## Outputs

- runtime_benchmark.csv with wall-clock time, core count (1 vs 4), and completion status (workflow.done present)
- Final ASV table, taxonomy table, and standard hand-off formats (biom, fasta, tab-separated tables)
- mock_community_relative_abundances.csv with observed per-taxon relative read abundances vs expected compositions
- Intermediary statistics figures/tables from dadasnake (read quality, chimera removal, taxonomy summaries via BASTA)

## How to apply

Run dadasnake from raw fastq files: write a YAML config (output_directory, raw_directory, sample_table path) and a tab-separated sample table with 'library', 'r1_file'/'r2_file' headers, validate with a dry-run (dadasnake -d -r config.yaml), then execute (e.g. dadasnake -l -n RUN1 -r config.yaml) limiting cpus via -t. For the runtime benchmark, run the 24-sample 16S V4 dataset (2.9 million reads, PRJNA517390) once with -t 1 (~150 min expected) and once with -t 4 after resetting the output directory (~100 min expected), confirming completion by the workflow.done marker. For ASV richness, process the 19-species fungal mock community with cutadapt primer removal (F-ITS4/R-fITS7; minimum overlap 10, max 2 primers per end, 0.2 mismatch rate), DADA2 error estimation/denoising and chimera removal, ITSx extraction (e-value 1.00E-05, target taxon F, minimum 1 region), mothur Bayesian classification against UNITE v8 (cutoff 60), and BASTA BLAST-hit summarization, then check that exactly 19 fungal ASVs are recovered (2 Aspergillus, 3 Fusarium strains, with the known Rhizophagus irregularis overestimation). For abundance benchmarking, aggregate the final ASV tables to taxa, normalize to relative read abundances, and join with the ground-truth mock compositions from Zenodo [60]; expect no correlation for the unequal prokaryotic community and multi-order-of-magnitude spread despite equal fungal input.

## Related tools

- **dadasnake** (Snakemake pipeline executing the full DADA2-based amplicon processing and mock community benchmarking runs) — https://github.com/a-h-b/dadasnake
- **DADA2** (Quality filtering, trimming, error estimation, denoising (ASV inference), and chimera removal) — http://benjjneb.github.io/dada2/
- **cutadapt** (Primer removal (and poly-G-tail removal) prior to DADA2 processing)
- **mothur** (Bayesian taxonomic classification of ASVs against UNITE/SILVA reference data (cutoff 60))
- **BLAST** (Sequence similarity search whose tabular hits are summarized with BASTA for taxonomic hand-off)
- **seqtk** (Optional downsampling of reads per sample to control dataset size in benchmark runs) — https://github.com/lh3/seqtk

## Examples

```
dadasnake -d -r config.yaml  # dry-run validation before benchmark runs (e.g. dadasnake -l -n RUN1 -r config.yaml -t 4)
```

## Evaluation signals

- The workflow.done marker appears after each dadasnake run, indicating successful completion of all Snakemake jobs
- Wall-clock runtime for the 24-sample 16S V4 dataset falls near 150 minutes on 1 core and drops to ~100 minutes on 4 cores
- The fungal mock community run recovers exactly 19 fungal ASVs with the expected strain structure (2 Aspergillus, 3 Fusarium) and the known Rhizophagus irregularis diversity overestimation
- The prokaryotic mock community shows no correlation between relative read abundance and the known unequal taxon composition
- mock_community_relative_abundances.csv covers all ground-truth taxa, with fungal relative abundances spanning several orders of magnitude despite equal input proportions

## Limitations

- Relative read abundances of ASVs misrepresent true taxon abundances: the prokaryotic mock community showed no correlation with expected composition, and equal-input fungal taxa spanned orders of magnitude in read abundance
- Strain misclassification can occur: 1 of the 3 Fusarium strains was misclassified despite micro-diversity being correctly identified
- Rhizophagus irregularis diversity is known to be overestimated in the fungal mock community processing
- Runtime results depend on available cores; the article benchmark used only 1 vs 4 cores on a small 24-sample dataset

## Evidence

- [abstract] The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi.: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
- [supplementary] Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60].: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [methods] error estimation & denoising - using DADA2, including Novaseq-enabled models: "error estimation & denoising - using DADA2, including Novaseq-enabled models"
- [readme] dadasnake is a Snakemake workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables, based on the DADA2 method.: "dadasnake is a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables, based on the"
