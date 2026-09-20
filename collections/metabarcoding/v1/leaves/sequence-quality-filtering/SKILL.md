---
name: sequence-quality-filtering
description: 'Use when starting from raw paired-end amplicon FASTQ reads and a per-sample
  feature table is the goal: reads must be demultiplexed by barcode, merged, and quality
  filtered before OTU clustering or ASV denoising.'
when_to_use_negative:
- Input is already a demultiplexed, quality-filtered read set or an existing OTU/ASV
  feature table — re-filtering would alter feature counts.
- The goal is shotgun metagenomic analysis; this skill covers the amplicon pipeline
  the paper recommends separately from metagenomic analyses.
- Reads are single-end or unpaired, so paired-end merging is not applicable.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Next-generation microbiome analysis pipeline implementing demultiplexing,
    quality filtering, and denoising/clustering functionality equivalent to DADA2/Deblur
    or UPARSE/USEARCH
- name: YongxinLiu/Liu2020ProteinCell
  role: Source of the pipeline scripts (demultiplexing, merging, quality filtering)
    used in the paper's reproducible amplicon analysis examples
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:018a61f2591745f3a943fe8f6c9145abb8a9f689eec56ca125df073d7b82c7dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - QIIME 2
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# sequence-quality-filtering

## Summary

Quality-control step of the recommended amplicon analysis pipeline (Liu et al. 2020, Protein & Cell) in which raw paired-end FASTQ reads are demultiplexed, merged, and quality filtered to retain high-quality sequences before OTU/ASV inference and feature table construction.

## When to use

Use when starting from raw paired-end amplicon FASTQ reads and a per-sample feature table is the goal: reads must be demultiplexed by barcode, merged, and quality filtered before OTU clustering or ASV denoising.

## When NOT to use

- Input is already a demultiplexed, quality-filtered read set or an existing OTU/ASV feature table — re-filtering would alter feature counts.
- The goal is shotgun metagenomic analysis; this skill covers the amplicon pipeline the paper recommends separately from metagenomic analyses.
- Reads are single-end or unpaired, so paired-end merging is not applicable.

## Inputs

- raw pooled paired-end amplicon FASTQ reads
- sample barcode sequences (demultiplexing index/barcode file)
- pipeline demultiplexing/quality-filtering scripts from the YongxinLiu/Liu2020ProteinCell repository

## Outputs

- per-sample demultiplexed FASTQ files
- merged, quality-filtered sequence reads for downstream OTU/ASV inference
- quality-filtered sequence set feeding the OTU/ASV feature table (otu_table.tsv)

## How to apply

Obtain the raw amplicon FASTQ reads and the pipeline scripts from the YongxinLiu/Liu2020ProteinCell repository accompanying the paper's step-by-step reproducible analysis guide. Demultiplex the pooled reads into per-sample FASTQ files using the sample barcode sequences via the pipeline's demultiplexing script. Merge paired-end reads and apply quality filtering to retain high-quality sequences for downstream inference — this filtered read set is the substrate for OTU clustering (e.g., UPARSE/USEARCH) or ASV denoising (DADA2/Deblur), with QIIME 2 providing equivalent denoising/clustering functionality. Only after filtering are representative sequences clustered or denoised, taxonomy assigned against a reference database, and the per-sample count table (otu_table.tsv) built, so aggressive or insufficient filtering directly changes feature counts.

## Related tools

- **QIIME 2** (Next-generation microbiome analysis pipeline implementing demultiplexing, quality filtering, and denoising/clustering functionality equivalent to DADA2/Deblur or UPARSE/USEARCH)
- **YongxinLiu/Liu2020ProteinCell** (Source of the pipeline scripts (demultiplexing, merging, quality filtering) used in the paper's reproducible amplicon analysis examples) — YongxinLiu/Liu2020ProteinCell

## Evaluation signals

- Per-sample FASTQ files exist after demultiplexing and sample counts match the barcode/sample metadata.
- Quality-filtered, merged read counts per sample are within expected ranges and reads pass the pipeline's quality criteria.
- Downstream OTU clustering (e.g., 97% UPARSE) or ASV denoising (DADA2/Deblur) runs successfully on the filtered reads.
- The final otu_table.tsv contains per-sample sequence counts and taxonomy columns consistent with the paper's step-by-step reproducible analysis guide.
- Results are reproducible using the scripts from the YongxinLiu/Liu2020ProteinCell repository.

## Limitations

- The paper recommends this pipeline specifically for amplicon data; metagenomic data require a different pipeline.
- Filtering thresholds affect downstream feature counts — over-filtering can drop samples/sequences, under-filtering lets low-quality sequences into OTU/ASV inference.
- The review provides guidance and sample code but does not name specific analysis software, pipelines, or versions in some sections; consult the reproducible guide repo for concrete parameters.

## Evidence

- [abstract] Abstract recommendation of pipelines: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [intro] QIIME 2 as the pipeline platform: "The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published (Bolyen et al., 2019)."
- [readme] Reproducible analysis repository README: "Pipeline &amp; Reproducible analysis examples for amplicon and metagenome in microbiome"
