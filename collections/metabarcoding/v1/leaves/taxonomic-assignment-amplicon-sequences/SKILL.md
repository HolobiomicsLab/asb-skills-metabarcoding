---
name: taxonomic-assignment-amplicon-sequences
description: Use when raw paired-end amplicon FASTQ reads must be demultiplexed, denoised/clustered,
  and converted into a taxonomy-assigned OTU/ASV feature table quantified per sample.
when_to_use_negative:
- Input is already a feature table (taxonomy assignment and count quantification are
  already done)
- Analysis target is shotgun metagenomic data — the paper treats the metagenomic pipeline
  separately from the amplicon pipeline
- Goal is downstream functional prediction (e.g., PICRUSt2) or tree visualization
  (e.g., table2itol/iTOL) rather than building the taxonomy-assigned feature table
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Reproducible, community-supported platform implementing the denoising (DADA2/Deblur-equivalent),
    clustering, and taxonomy-assignment steps of the amplicon pipeline
- name: YongxinLiu/Liu2020ProteinCell pipeline scripts
  role: Provides the demultiplexing, merge/quality-filter, OTU/ASV inference, and
    feature-table construction scripts from the paper's step-by-step reproducible
    analysis guide
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:2ce762a34f900aa4637855ba31ba16ca236d43ed6566de930ad739dd42bbe80f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - QIIME 2
  - YongxinLiu/Liu2020ProteinCell pipeline scripts
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# taxonomic-assignment-amplicon-sequences

## Summary

Reproduces the Liu et al. 2020 amplicon pipeline step that assigns taxonomy to OTU/ASV representative sequences and builds the per-sample count feature table (otu_table.tsv) from raw paired-end FASTQ reads, following the paper's step-by-step reproducible analysis guide.

## When to use

Use when raw paired-end amplicon FASTQ reads must be demultiplexed, denoised/clustered, and converted into a taxonomy-assigned OTU/ASV feature table quantified per sample.

## When NOT to use

- Input is already a feature table (taxonomy assignment and count quantification are already done)
- Analysis target is shotgun metagenomic data — the paper treats the metagenomic pipeline separately from the amplicon pipeline
- Goal is downstream functional prediction (e.g., PICRUSt2) or tree visualization (e.g., table2itol/iTOL) rather than building the taxonomy-assigned feature table

## Inputs

- Raw paired-end amplicon sequencing FASTQ reads (pooled, with sample barcodes)
- Sample barcode/metadata mapping for demultiplexing
- Pipeline scripts from the YongxinLiu/Liu2020ProteinCell repository
- Reference taxonomy database for feature annotation

## Outputs

- otu_table.tsv — per-sample OTU/ASV sequence count feature table with taxonomy columns
- Representative OTU/ASV sequences (UPARSE 97% OTUs or DADA2/Deblur exact ASVs)

## How to apply

Obtain the raw amplicon FASTQ reads and the pipeline scripts from the YongxinLiu/Liu2020ProteinCell repository accompanying the paper's reproducible analysis guide. Demultiplex the pooled reads into per-sample FASTQ files using sample barcodes, then merge paired-end reads and apply quality filtering to retain high-quality sequences. Cluster quality-filtered sequences into OTUs with UPARSE/USEARCH at 97% similarity, or infer exact ASVs with DADA2/Deblur (QIIME 2 implements the equivalent denoising/clustering functionality). Assign taxonomy to each OTU/ASV against a reference database, quantify feature sequence frequencies per sample, and write the final feature table with taxonomy assignments to otu_table.tsv.

## Related tools

- **QIIME 2** (Reproducible, community-supported platform implementing the denoising (DADA2/Deblur-equivalent), clustering, and taxonomy-assignment steps of the amplicon pipeline)
- **YongxinLiu/Liu2020ProteinCell pipeline scripts** (Provides the demultiplexing, merge/quality-filter, OTU/ASV inference, and feature-table construction scripts from the paper's step-by-step reproducible analysis guide) — YongxinLiu/Liu2020ProteinCell

## Evaluation signals

- otu_table.tsv exists and contains per-sample sequence counts plus taxonomy columns
- Representative sequences were derived either by UPARSE clustering at 97% similarity or by denoising into exact ASVs (DADA2/Deblur), not by ad hoc clustering
- Every OTU/ASV in the feature table carries a taxonomy assignment from a reference database
- Per-sample counts in the feature table are consistent with the demultiplexed, quality-filtered read counts of each sample
- Results are reproducible and consistent with the paper's step-by-step reproducible analysis guide

## Limitations

- The paper notes that new developments make it challenging for researchers, especially those without a bioinformatics background, to choose suitable software and pipelines — tool choice (OTU clustering vs. ASV denoising) is a decision point that affects the final table
- Pipeline scripts and raw reads must be obtained from the accompanying GitHub repository; without them the step-by-step reproduction cannot be executed as described
- The extracted article text does not name specific reference databases or software versions for taxonomy assignment, so those details must be confirmed against the full reproducible guide

## Evidence

- [abstract] Recommended pipelines for amplicon analysis: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [abstract] Step-by-step reproducible guide: "Finally, a step-by-step reproducible analysis guide is introduced."
- [intro] QIIME 2 as the platform: "The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published (Bolyen et al., 2019)."
- [readme] Accompanying pipeline repository: "Pipeline &amp; Reproducible analysis examples for amplicon and metagenome in microbiome"
