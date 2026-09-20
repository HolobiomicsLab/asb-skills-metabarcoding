---
name: metagenomic-read-quality-control
description: Use when raw metagenomic sequencing FASTQ files must be converted into
  clean, host-free reads before taxonomic or functional profiling, or when reproducing
  the Liu 2020 metagenomic pipeline.
when_to_use_negative:
- Input is already a cleaned, non-host read set or an assembled contig set rather
  than raw metagenomic FASTQ
- The analysis goal is 16S/amplicon processing (the review recommends a separate amplicon
  pipeline, e.g., QIIME 2, for that data type)
- Downstream feature tables (taxonomic profiles or KEGG orthology tables) already
  exist and only profiling or statistics are needed
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: KneadData
  role: 'Primary QC pipeline: chains Trimmomatic and Bowtie 2 to trim adapters/low-quality
    bases and remove host reads from metagenomic FASTQ'
- name: Trimmomatic
  role: Adapter and quality trimming of raw FASTQ reads, integrated within KneadData
    or run standalone
- name: Bowtie2
  role: Aligns reads against the host reference so host-contaminating reads can be
    removed
- name: YongxinLiu/Liu2020ProteinCell
  role: Reproducible analysis scripts implementing the recommended metagenomic QC-to-profiling
    workflow
  repo: https://github.com/YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:ef0453ebf6e8c1f7408ead1060a0a0cb00bd73318929e43b986146fc5b3618a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - Bowtie2
  - HUMAnN2
  - KneadData
  - Kraken2
  - MetaPhlAn2
  - Trimmomatic
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# metagenomic-read-quality-control

## Summary

Quality-control step of the metagenomic pipeline recommended by Liu et al. 2020: raw metagenomic FASTQ reads are adapter/quality trimmed with Trimmomatic and host-aligned reads are removed with Bowtie 2, chained together by KneadData, to yield clean non-host reads for downstream taxonomic (MetaPhlAn2, Kraken2) and functional (HUMAnN2) profiling.

## When to use

Use when raw metagenomic sequencing FASTQ files must be converted into clean, host-free reads before taxonomic or functional profiling, or when reproducing the Liu 2020 metagenomic pipeline.

## When NOT to use

- Input is already a cleaned, non-host read set or an assembled contig set rather than raw metagenomic FASTQ
- The analysis goal is 16S/amplicon processing (the review recommends a separate amplicon pipeline, e.g., QIIME 2, for that data type)
- Downstream feature tables (taxonomic profiles or KEGG orthology tables) already exist and only profiling or statistics are needed

## Inputs

- Raw paired-end metagenomic sequencing FASTQ files
- Host reference genome/index for Bowtie 2 alignment (e.g., human)

## Outputs

- Cleaned, non-host metagenomic FASTQ files (adapter- and quality-trimmed, host reads removed)
- QC log/report files from KneadData (Trimmomatic and Bowtie 2 per-sample statistics)

## How to apply

Obtain the reproducible analysis scripts from the GitHub repository YongxinLiu/Liu2020ProteinCell accompanying the step-by-step guide. Run KneadData, which chains Trimmomatic adapter/quality trimming with Bowtie 2 alignment against the host reference, on the raw paired-end metagenomic FASTQ files to remove host-contaminating reads. The cleaned non-host FASTQ output becomes the input for MetaPhlAn2 and Kraken2 (taxonomic profiles) and HUMAnN2 (functional profiles regressed to KEGG orthology tables). The rationale is that host-read contamination and low-quality bases would otherwise bias taxonomic and functional abundance estimates; KneadData is recommended because it integrates both trimming and host-depletion in one pipeline. Decision point: if KneadData is unavailable, the review notes Trimmomatic + Bowtie 2 run separately as the equivalent alternative.

## Related tools

- **KneadData** (Primary QC pipeline: chains Trimmomatic and Bowtie 2 to trim adapters/low-quality bases and remove host reads from metagenomic FASTQ)
- **Trimmomatic** (Adapter and quality trimming of raw FASTQ reads, integrated within KneadData or run standalone)
- **Bowtie2** (Aligns reads against the host reference so host-contaminating reads can be removed)
- **YongxinLiu/Liu2020ProteinCell** (Reproducible analysis scripts implementing the recommended metagenomic QC-to-profiling workflow) — https://github.com/YongxinLiu/Liu2020ProteinCell

## Examples

```
kneaddata --input sample_R1.fastq --input sample_R2.fastq --reference-db human --output kneaddata_out
```

## Evaluation signals

- The section text reports no quantitative acceptance criterion: confirm the run succeeds by checking that cleaned read files exist, are non-empty, and correspond to each analyzed sample
- Paired-end inputs remain synchronized after QC (R1/R2 output pairs contain equal read counts)
- Host-read removal is evidenced by reads discarded in the Bowtie 2 host-alignment stage reported in the KneadData log
- The cleaned reads support the downstream steps: MetaPhlAn2/Kraken2 taxonomic tables and HUMAnN2 KEGG orthology (KO) tables are produced, non-empty, and contain rows for the analyzed samples

## Limitations

- The review's section text reports no quantitative acceptance criterion (e.g., minimum surviving read count or host-read percentage) for QC success; validation is limited to existence/non-emptiness of outputs
- Host reference choice and index quality directly determine host-read removal effectiveness; an inappropriate reference leaves host contamination
- Metatranscriptome, metaproteome, and metabolome analyses are out of scope — the review notes these methods were 'not been widely accepted due to their high cost and the complex experimental and analysis methods required'

## Evidence

- [other] KneadData integrates Trimmomatic and Bowtie 2 for QC: [evidence span withheld — anchor status: not_found]
- [abstract] Review recommends specific pipelines for amplicon and metagenomic analyses: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [readme] Liu2020ProteinCell repository provides reproducible analysis examples: "Pipeline &amp; Reproducible analysis examples for amplicon and metagenome in microbiome"
- [abstract] Step-by-step reproducible analysis guide is introduced: "Finally, a step-by-step reproducible analysis guide is introduced."
