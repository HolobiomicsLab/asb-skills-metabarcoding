---
name: marker-gene-taxonomic-profiling
description: Use when raw metagenomic sequencing reads (FASTQ) from microbiome samples
  must be converted into per-sample taxonomic abundance profiles and functional gene-family/KO
  tables for downstream statistical analysis.
when_to_use_negative:
- Input is amplicon (e.g. 16S) data — the article recommends a separate amplicon analysis
  pipeline (QIIME 2-based) rather than this metagenomic workflow
- Input is already a taxonomic or KO feature table — proceed directly to the article's
  statistical/visualization methods instead of re-profiling
- Data are metatranscriptome, metaproteome, or metabolome — the article notes these
  'have not been widely accepted due to their high cost and the complex experimental
  and analysis methods required'
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: KneadData
  role: 'Quality control and host-read removal: chains Trimmomatic trimming with Bowtie2
    host alignment to output clean reads for profiling'
- name: Trimmomatic
  role: Adapter/quality trimming of raw FASTQ reads, invoked within the KneadData
    pipeline
- name: Bowtie2
  role: Aligns trimmed reads against the host reference so host-contaminating reads
    can be removed
- name: MetaPhlAn2
  role: Marker-gene-based taxonomic profiler run on cleaned non-host reads to generate
    taxonomic abundance profiles
- name: Kraken2
  role: k-mer-based taxonomic classifier run on cleaned reads as a complementary taxonomic
    profile
- name: HUMAnN2
  role: Produces functional profiles from cleaned reads and regresses gene families
    to KEGG orthology (KO) abundance tables
- name: Liu2020ProteinCell scripts
  role: Step-by-step reproducible analysis scripts that implement the full pipeline
    end to end
  repo: https://github.com/YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:0d64c4df26ccff328b849ca3c5fec688571eb171439012f381a08a6478bc2ccb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - Bowtie2
  - HUMAnN2
  - KneadData
  - Kraken2
  - Liu2020ProteinCell scripts
  - MetaPhlAn2
  - Trimmomatic
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# marker-gene-taxonomic-profiling

## Summary

A reproducible metagenomic profiling pipeline from the Liu et al. 2020 Protein & Cell practical guide that converts raw metagenomic FASTQ reads into taxonomic abundance tables (via marker-gene and k-mer profilers) and KEGG orthology (KO) functional tables, using the step-by-step scripts from the YongxinLiu/Liu2020ProteinCell repository.

## When to use

Use when raw metagenomic sequencing reads (FASTQ) from microbiome samples must be converted into per-sample taxonomic abundance profiles and functional gene-family/KO tables for downstream statistical analysis.

## When NOT to use

- Input is amplicon (e.g. 16S) data — the article recommends a separate amplicon analysis pipeline (QIIME 2-based) rather than this metagenomic workflow
- Input is already a taxonomic or KO feature table — proceed directly to the article's statistical/visualization methods instead of re-profiling
- Data are metatranscriptome, metaproteome, or metabolome — the article notes these 'have not been widely accepted due to their high cost and the complex experimental and analysis methods required'

## Inputs

- Raw metagenomic sequencing FASTQ files (paired-end reads)
- Host reference genome/index for Bowtie2 host-read removal

## Outputs

- Taxonomic abundance profile tables from MetaPhlAn2
- Taxonomic abundance profile tables from Kraken2
- Functional gene-family profiles from HUMAnN2
- KEGG orthology (KO) abundance tables regressed from HUMAnN2 gene families

## How to apply

Obtain the step-by-step reproducible analysis scripts from the GitHub repository YongxinLiu/Liu2020ProteinCell accompanying the guide. First clean raw metagenomic FASTQ files by running KneadData, which chains Trimmomatic adapter/quality trimming with Bowtie2 alignment against the host reference to remove host-contaminating reads. Then run MetaPhlAn2 (marker-gene based) and Kraken2 (k-mer based) on the cleaned non-host reads to generate taxonomic abundance profiles, providing two complementary taxonomic estimates. Run HUMAnN2 on the same cleaned reads to produce functional profiles and regress gene families to KEGG orthology (KO) abundance tables. Collect the taxonomic profile tables and KO functional tables as final outputs; since the article reports no quantitative acceptance criterion, validate by checking that output tables exist, are non-empty, and contain rows for the analyzed samples.

## Related tools

- **KneadData** (Quality control and host-read removal: chains Trimmomatic trimming with Bowtie2 host alignment to output clean reads for profiling)
- **Trimmomatic** (Adapter/quality trimming of raw FASTQ reads, invoked within the KneadData pipeline)
- **Bowtie2** (Aligns trimmed reads against the host reference so host-contaminating reads can be removed)
- **MetaPhlAn2** (Marker-gene-based taxonomic profiler run on cleaned non-host reads to generate taxonomic abundance profiles)
- **Kraken2** (k-mer-based taxonomic classifier run on cleaned reads as a complementary taxonomic profile)
- **HUMAnN2** (Produces functional profiles from cleaned reads and regresses gene families to KEGG orthology (KO) abundance tables)
- **Liu2020ProteinCell scripts** (Step-by-step reproducible analysis scripts that implement the full pipeline end to end) — https://github.com/YongxinLiu/Liu2020ProteinCell

## Examples

```
git clone https://github.com/YongxinLiu/Liu2020ProteinCell && bash run_pipeline.sh  # KneadData -> MetaPhlAn2/Kraken2 -> HUMAnN2 per the reproducible guide
```

## Evaluation signals

- Taxonomic profile tables and KEGG-orthology functional tables exist, are non-empty, and contain rows for every analyzed sample (the article's stated validation check)
- Cleaned read counts are lower than raw input reads, confirming Trimmomatic trimming and Bowtie2 host-read removal executed
- Both MetaPhlAn2 and Kraken2 outputs are produced and report taxa for the analyzed samples, allowing cross-profiler comparison
- HUMAnN2 gene-family output can be regressed to KO identifiers, and the resulting KO table has valid KEGG orthology row identifiers

## Limitations

- The article's section text reports no quantitative acceptance criterion for the pipeline run, so success is judged only by output-table presence, non-emptiness, and per-sample rows
- Host contamination must be removed first (KneadData/Trimmomatic+Bowtie2); profiling uncleaned reads would skew taxonomic and functional estimates
- Two taxonomic profilers (MetaPhlAn2 and Kraken2) may give differing abundance estimates; the article offers no reconciliation rule, so results should be reported per tool

## Evidence

- [abstract] Recommended metagenomic pipeline: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases"
- [abstract] Step-by-step reproducible analysis: "Finally, a step-by-step reproducible analysis guide is introduced."
- [readme] Reproducible analysis repository: "Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome"
- [discussion] Analysis scope discussion: "from the selection of sequencing methods, analysis software/pipelines, statistical analysis and visualization to the implementation of reproducible analysis"
