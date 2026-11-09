---
name: reproducible-pipeline-script-execution
description: Use when reproducing the review's recommended metagenomic workflow from
  raw sequencing FASTQ files into taxonomic abundance tables and KO functional profile
  tables using its companion scripts.
when_to_use_negative:
- Input is already a taxonomic or KO feature table — the pipeline consumes raw FASTQ,
  so re-running KneadData/HUMAnN2 on processed tables is wrong
- The analysis goal is metatranscriptome, metaproteome, or metabolome analysis, which
  the review notes are 'not widely accepted due to their high cost and the complex
  experimental and analysis methods required' and are not covered by this pipeline
- Only statistical analysis or visualization of existing microbiome results is needed
  (e.g. alpha/beta diversity, networks, machine learning) rather than feature-table
  generation from reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: KneadData
  role: 'Quality control and host-read removal: chains Trimmomatic trimming with Bowtie2
    host alignment to output clean reads'
- name: Trimmomatic
  role: Adapter/quality trimming step invoked within the KneadData pipeline
- name: Bowtie2
  role: Host-reference alignment used to remove host-contaminating reads within KneadData
- name: MetaPhlAn2
  role: Generates taxonomic abundance profiles from cleaned non-host reads
- name: Kraken2
  role: Second taxonomic profiler run on cleaned reads
- name: HUMAnN2
  role: Produces functional profiles and regresses gene families to KEGG orthology
    (KO) abundance tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:6df9e26f47ffce34e40cb88250d0cee7f658d2b3867e442a00e94009c7f1d74a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie2
  - HUMAnN2
  - KneadData
  - Kraken2
  - MetaPhlAn2
  - Trimmomatic
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# reproducible-pipeline-script-execution

## Summary

Executes the paper's step-by-step reproducible analysis scripts (YongxinLiu/Liu2020ProteinCell) to run the recommended metagenomic pipeline — KneadData cleaning, MetaPhlAn2/Kraken2 taxonomy, HUMAnN2 functional profiling — turning raw FASTQ into taxonomic and KEGG-orthology (KO) feature tables.

## When to use

Use when reproducing the review's recommended metagenomic workflow from raw sequencing FASTQ files into taxonomic abundance tables and KO functional profile tables using its companion scripts.

## When NOT to use

- Input is already a taxonomic or KO feature table — the pipeline consumes raw FASTQ, so re-running KneadData/HUMAnN2 on processed tables is wrong
- The analysis goal is metatranscriptome, metaproteome, or metabolome analysis, which the review notes are 'not widely accepted due to their high cost and the complex experimental and analysis methods required' and are not covered by this pipeline
- Only statistical analysis or visualization of existing microbiome results is needed (e.g. alpha/beta diversity, networks, machine learning) rather than feature-table generation from reads

## Inputs

- Raw metagenomic sequencing FASTQ files (paired-end)
- Host reference genome/index for Bowtie2-based host-read removal
- Step-by-step reproducible analysis scripts from the YongxinLiu/Liu2020ProteinCell repository

## Outputs

- Cleaned non-host metagenomic reads (KneadData output)
- Taxonomic abundance profile tables (MetaPhlAn2 and Kraken2)
- Functional profile tables with gene families regressed to KEGG orthology (KO) abundance tables (HUMAnN2)

## How to apply

First obtain the step-by-step reproducible analysis scripts from the GitHub repository YongxinLiu/Liu2020ProteinCell accompanying the guide. Run KneadData, which chains Trimmomatic adapter/quality trimming with Bowtie2 alignment against the host reference, to clean raw metagenomic FASTQ files and remove host-contaminating reads. Run MetaPhlAn2 and Kraken2 on the cleaned non-host reads to generate taxonomic abundance profiles, then run HUMAnN2 to produce functional profiles and regress gene families to KEGG orthology (KO) abundance tables. The rationale is that the review recommends this specific pipeline for metagenomic analyses and provides sample codes so researchers can implement reproducible analysis end-to-end. Because the section text reports no quantitative acceptance criterion, confirm success by checking that the taxonomic and KO output tables exist, are non-empty, and contain rows for the analyzed samples.

## Related tools

- **KneadData** (Quality control and host-read removal: chains Trimmomatic trimming with Bowtie2 host alignment to output clean reads)
- **Trimmomatic** (Adapter/quality trimming step invoked within the KneadData pipeline)
- **Bowtie2** (Host-reference alignment used to remove host-contaminating reads within KneadData)
- **MetaPhlAn2** (Generates taxonomic abundance profiles from cleaned non-host reads)
- **Kraken2** (Second taxonomic profiler run on cleaned reads)
- **HUMAnN2** (Produces functional profiles and regresses gene families to KEGG orthology (KO) abundance tables)

## Examples

```
git clone https://github.com/YongxinLiu/Liu2020ProteinCell
```

## Evaluation signals

- KneadData run produces cleaned non-host FASTQ output files for every input sample
- MetaPhlAn2 and Kraken2 taxonomic profile tables exist, are non-empty, and contain rows for the analyzed samples
- HUMAnN2 output includes KEGG-orthology (KO) abundance tables rather than only raw gene-family tables
- All steps were executed via the scripts from the YongxinLiu/Liu2020ProteinCell repository, matching the step-by-step guide
- Final deliverables are both a taxonomic profile table and a KO functional table per the pipeline's stated outputs

## Limitations

- The section text reports no quantitative acceptance criterion, so validation relies on output existence, non-emptiness, and sample coverage rather than numeric thresholds
- Metatranscriptome, metaproteome, and metabolome analysis are out of scope: the review notes these methods 'have not been widely accepted due to their high cost and the complex experimental and analysis methods required'
- Host-read removal quality depends on the host reference supplied to Bowtie2; an inappropriate host index will leave contaminating reads in downstream profiles

## Evidence

- [abstract] Finally, a step-by-step reproducible analysis guide is introduced.: "Finally, a step-by-step reproducible analysis guide is introduced."
- [intro] Review provides sample codes and suggestions: "In this review, we discuss the widely used software packages for microbiome analyses, summarize their advantages and limitations, and provide sample codes"
- [readme] Liu2020ProteinCell README describes the repo: "Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome"
- [abstract] Review recommends specific pipelines: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
