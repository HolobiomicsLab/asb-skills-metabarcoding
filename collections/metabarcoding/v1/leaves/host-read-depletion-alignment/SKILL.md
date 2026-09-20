---
name: host-read-depletion-alignment
description: Use when raw metagenomic FASTQ files must be cleaned of host-contaminating
  reads before taxonomic (MetaPhlAn2, Kraken2) or functional (HUMAnN2) profiling.
when_to_use_negative:
- Input data are amplicon (e.g., 16S rRNA gene) reads — the review treats amplicon
  and metagenomic pipelines as distinct, and host depletion via Bowtie2 alignment
  is a shotgun-metagenome step
- Reads have already passed quality control and host-read removal and you only need
  taxonomic or functional profiling (proceed directly to MetaPhlAn2/Kraken2 or HUMAnN2)
- You need to generate feature tables themselves rather than the clean reads that
  feed feature-table generation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: KneadData
  role: QC and host-read removal wrapper that chains Trimmomatic trimming with Bowtie2
    host alignment
- name: Trimmomatic
  role: Adapter and quality trimming of raw metagenomic FASTQ reads (used within KneadData)
- name: Bowtie2
  role: Alignment of trimmed reads against the host reference to identify and remove
    host-contaminating reads
- name: YongxinLiu/Liu2020ProteinCell
  role: Reproducible analysis scripts implementing the recommended metagenomic pipeline
    including the host-depletion step
  repo: https://github.com/YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:d1094967fee4b62abaa7f9b88198348f0e4599e62895a9087eb0379460fea88b
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# host-read-depletion-alignment

## Summary

Quality control and host-read removal step of the recommended metagenomic pipeline, implemented with KneadData, which chains Trimmomatic adapter/quality trimming with Bowtie2 alignment against a host reference to produce clean non-host reads for taxonomic and functional profiling.

## When to use

Use when raw metagenomic FASTQ files must be cleaned of host-contaminating reads before taxonomic (MetaPhlAn2, Kraken2) or functional (HUMAnN2) profiling.

## When NOT to use

- Input data are amplicon (e.g., 16S rRNA gene) reads — the review treats amplicon and metagenomic pipelines as distinct, and host depletion via Bowtie2 alignment is a shotgun-metagenome step
- Reads have already passed quality control and host-read removal and you only need taxonomic or functional profiling (proceed directly to MetaPhlAn2/Kraken2 or HUMAnN2)
- You need to generate feature tables themselves rather than the clean reads that feed feature-table generation

## Inputs

- Raw paired-end metagenomic sequencing FASTQ files
- Host reference genome / Bowtie2 index for host-read alignment

## Outputs

- Clean, non-host reads (FASTQ) ready for MetaPhlAn2, Kraken2, and HUMAnN2 profiling
- Trimming and host-removal run reports from KneadData

## How to apply

Run KneadData on raw paired-end metagenomic FASTQ files; it internally chains Trimmomatic (adapter and quality trimming) with Bowtie2 alignment against the host reference genome, discarding reads that align to the host. Obtain the step-by-step reproducible analysis scripts from the GitHub repository YongxinLiu/Liu2020ProteinCell accompanying the review's reproducible analysis guide. The output clean, non-host reads then feed directly into MetaPhlAn2 and Kraken2 for taxonomic abundance profiles and HUMAnN2 for functional profiles regressed to KEGG orthology (KO) tables. Because the section text reports no quantitative acceptance criterion, judge success by confirming that clean-read FASTQ outputs exist, are non-empty, and that downstream taxonomic and KO tables contain rows for the analyzed samples.

## Related tools

- **KneadData** (QC and host-read removal wrapper that chains Trimmomatic trimming with Bowtie2 host alignment)
- **Trimmomatic** (Adapter and quality trimming of raw metagenomic FASTQ reads (used within KneadData))
- **Bowtie2** (Alignment of trimmed reads against the host reference to identify and remove host-contaminating reads)
- **YongxinLiu/Liu2020ProteinCell** (Reproducible analysis scripts implementing the recommended metagenomic pipeline including the host-depletion step) — https://github.com/YongxinLiu/Liu2020ProteinCell

## Examples

```
kneaddata -i1 sample_R1.fastq.gz -i2 sample_R2.fastq.gz -db bowtie2_host_index -o kneaddata_out/
```

## Evaluation signals

- Clean-read FASTQ outputs exist and are non-empty for every analyzed sample
- Downstream taxonomic profile tables (MetaPhlAn2, Kraken2) are non-empty and contain rows for all analyzed samples
- HUMAnN2 output regresses to a KEGG orthology (KO) abundance table with rows for the analyzed samples
- Read counts after host removal are lower than raw input counts, consistent with trimming plus host-read discarding
- Run scripts match the step-by-step reproducible workflow in the YongxinLiu/Liu2020ProteinCell repository

## Limitations

- The section text reports no quantitative acceptance criterion; validation is limited to checking that output tables exist, are non-empty, and contain rows for the analyzed samples
- Host removal quality depends on the choice and completeness of the host reference genome used for Bowtie2 alignment
- KneadData is offered as the recommended route but the review notes the equivalent alternative of running Trimmomatic + Bowtie 2 directly

## Evidence

- [methods] Recommended pipeline start: [evidence span withheld — anchor status: not_found]
- [other] KneadData mechanics: [evidence span withheld — anchor status: not_found]
- [readme] Reproducible repo description: "Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome"
