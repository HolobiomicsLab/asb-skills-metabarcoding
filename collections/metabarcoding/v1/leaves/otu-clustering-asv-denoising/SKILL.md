---
name: otu-clustering-asv-denoising
description: Use when the input is raw paired-end amplicon FASTQ reads plus sample
  barcode metadata and the goal is a per-sample OTU/ASV count table with taxonomy
  (e.g., otu_table.tsv) for downstream diversity or composition analysis.
when_to_use_negative:
- The input is already an OTU/ASV feature table (this skill produces one, not consumes
  one)
- The data are shotgun metagenomic reads rather than 16S/amplicon reads — use the
  paper's recommended metagenomic analysis pipeline instead
- Only downstream statistics (alpha/beta diversity, composition, correlation) are
  needed; this skill covers only read-to-feature-table inference
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Primary pipeline implementing denoising (ASV inference) and clustering (OTU
    picking) of amplicon reads; recommended platform in the article
- name: UPARSE/USEARCH
  role: Clusters quality-filtered merged reads into OTUs at 97% similarity as one
    branch of the feature-inference decision point
- name: DADA2/Deblur
  role: Infers exact ASVs from quality-filtered reads as the alternative to OTU clustering
- name: YongxinLiu/Liu2020ProteinCell
  role: Provides the pipeline scripts and reproducible analysis examples used to run
    demultiplexing through feature-table generation
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:5c09d4c9e8a6bfbc4235840cb5727d38943930543f0f4f6bf4726db7d30e54bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - DADA2/Deblur
  - QIIME 2
  - UPARSE/USEARCH
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# otu-clustering-asv-denoising

## Summary

Turns raw demultiplexed paired-end amplicon FASTQ reads into a quantified OTU/ASV feature table with taxonomy, following the amplicon pipeline recommended by Liu et al. (2020) and demonstrated in the YongxinLiu/Liu2020ProteinCell reproducible-analysis guide.

## When to use

Use when the input is raw paired-end amplicon FASTQ reads plus sample barcode metadata and the goal is a per-sample OTU/ASV count table with taxonomy (e.g., otu_table.tsv) for downstream diversity or composition analysis.

## When NOT to use

- The input is already an OTU/ASV feature table (this skill produces one, not consumes one)
- The data are shotgun metagenomic reads rather than 16S/amplicon reads — use the paper's recommended metagenomic analysis pipeline instead
- Only downstream statistics (alpha/beta diversity, composition, correlation) are needed; this skill covers only read-to-feature-table inference

## Inputs

- Raw paired-end amplicon FASTQ reads (pooled or per-run)
- Sample barcode sequences / demultiplexing mapping metadata
- Pipeline scripts from the YongxinLiu/Liu2020ProteinCell repository
- Reference taxonomy database for OTU/ASV assignment

## Outputs

- otu_table.tsv — per-sample OTU/ASV sequence counts with taxonomy columns
- Representative OTU/ASV sequences (from UPARSE 97% clustering or DADA2/Deblur denoising)

## How to apply

Obtain the raw amplicon FASTQ reads and the pipeline scripts from the YongxinLiu/Liu2020ProteinCell repository that accompanies the paper's step-by-step reproducible analysis guide. Demultiplex the pooled reads into per-sample FASTQ files using the sample barcode sequences with the repository's demultiplexing script, then merge paired-end reads and apply quality filtering to retain high-quality sequences for inference. At the decision point between feature inference strategies, either cluster quality-filtered sequences into OTUs at 97% similarity with UPARSE/USEARCH, or infer exact ASVs with DADA2/Deblur (QIIME 2 implements equivalent denoising/clustering functionality). Assign taxonomy to each OTU/ASV against a reference database and quantify feature sequence frequencies per sample to build the count table, writing the final feature table with taxonomy assignments to otu_table.tsv.

## Related tools

- **QIIME 2** (Primary pipeline implementing denoising (ASV inference) and clustering (OTU picking) of amplicon reads; recommended platform in the article)
- **UPARSE/USEARCH** (Clusters quality-filtered merged reads into OTUs at 97% similarity as one branch of the feature-inference decision point)
- **DADA2/Deblur** (Infers exact ASVs from quality-filtered reads as the alternative to OTU clustering)
- **YongxinLiu/Liu2020ProteinCell** (Provides the pipeline scripts and reproducible analysis examples used to run demultiplexing through feature-table generation) — YongxinLiu/Liu2020ProteinCell

## Evaluation signals

- otu_table.tsv exists and contains per-sample sequence counts plus taxonomy columns
- The feature table is consistent with the results of the paper's step-by-step reproducible analysis guide
- Each OTU/ASV traces back to either 97%-similarity UPARSE clustering or exact denoised ASVs (DADA2/Deblur) from quality-filtered merged reads
- Demultiplexed per-sample FASTQ files are recoverable from the barcode sequences before merging/quality filtering
- Row counts (features) and column counts (samples) in otu_table.tsv match the sample metadata used for demultiplexing

## Limitations

- The choice between OTU clustering (97% UPARSE) and ASV denoising (DADA2/Deblur) changes feature definitions; results are not interchangeable across runs
- Quality filtering thresholds decide which sequences are retained for inference — overly strict/loose settings shift feature counts
- The paper notes it is challenging for researchers, especially those without a bioinformatics background, to choose suitable software and pipelines; tool selection should follow its recommendations
- Metatranscriptome/metaproteome/metabolome-style extensions are out of scope; the article flags those methods as not widely accepted due to high cost and complex experimental and analysis methods

## Evidence

- [abstract] Specific pipelines are recommended for amplicon and metagenomic analyses: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [intro] QIIME 2 recommendation: "The next-generation microbiome analysis pipeline QIIME 2, a reproducible, interactive, efficient, community-supported platform was recently published (Bolyen et al., 2019)."
- [abstract] Step-by-step reproducible analysis guide: "Finally, a step-by-step reproducible analysis guide is introduced."
- [readme] Repository purpose: "Pipeline &amp; Reproducible analysis examples for amplicon and metagenome in microbiome"
