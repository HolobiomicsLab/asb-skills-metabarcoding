---
name: de-novo-metagenome-assembly
description: Use when quality-filtered shotgun metagenomic reads (*_1.fastq.gz / *_2.fastq.gz
  pairs) must be assembled into contigs without a reference genome, to enable coverage-based
  binning with MetaBAT/MaxBin or functional annotation.
when_to_use_negative:
- Input is amplicon (16S rRNA) data — use the amplicon_analysis.sh workflow (mothur/dada2
  with OTU clustering or ASV denoising) instead of metagenomic assembly
- A suitable reference-based route suffices (kraken2/MetaPhlAn2/DIAMOND vs NR, HUMAnN2)
  and no contigs or MAGs are needed — reference-based profiling does not require assembly
- Input reads are not demultiplexed paired-end FASTQ in the required *.fastq.gz naming
  scheme; the workflow expects *_1.fastq.gz and *_2.fastq.gz pairs
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3168
tools:
- name: MEGAHIT
  role: assembler for single-sample and co-assembly of quality-controlled metagenomic
    reads
- name: SPAdes
  role: assembler for single-sample and co-assembly of quality-controlled metagenomic
    reads
- name: IDBA
  role: third integrated assembler used for single-sample assembly in the workflow
- name: BBMap
  role: later backmaps filtered contigs to their reads to generate sorted BAM and
    depth files for coverage-based binning
- name: Trimmomatic
  role: upstream adapter, low-quality-read and host-contamination removal before assembly
- name: Sickle
  role: upstream low-quality read trimming before assembly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:f69d734f71cfa01a0d64496ed88f8148251522f12ac53de56a2f9d4e96e84b94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBMap
  - CheckM
  - DIAMOND
  - FastQC
  - HUMAnN2
  - IDBA
  - MEGAHIT
  - MEGAN
  - MaxBin
  - MetaBAT
  - MetaPhlAn2
  - Prokka
  - SPAdes
  - Sickle
  - Trimmomatic
  - kraken2
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# de-novo-metagenome-assembly

## Summary

Assembles quality-controlled paired-end metagenomic reads (e.g. metaHIT gut survey FASTQ) into contigs de novo using three integrated assemblers (MEGAHIT, SPAdes, IDBA) for single-sample and co-assembly, then filters contigs to a minimum length of 1000 bp before downstream binning and profiling.

## When to use

Use when quality-filtered shotgun metagenomic reads (*_1.fastq.gz / *_2.fastq.gz pairs) must be assembled into contigs without a reference genome, to enable coverage-based binning with MetaBAT/MaxBin or functional annotation.

## When NOT to use

- Input is amplicon (16S rRNA) data — use the amplicon_analysis.sh workflow (mothur/dada2 with OTU clustering or ASV denoising) instead of metagenomic assembly
- A suitable reference-based route suffices (kraken2/MetaPhlAn2/DIAMOND vs NR, HUMAnN2) and no contigs or MAGs are needed — reference-based profiling does not require assembly
- Input reads are not demultiplexed paired-end FASTQ in the required *.fastq.gz naming scheme; the workflow expects *_1.fastq.gz and *_2.fastq.gz pairs

## Inputs

- quality-filtered, demultiplexed, gzip-compressed paired-end Illumina reads named *_1.fastq.gz and *_2.fastq.gz (e.g. metaHIT gut survey data from ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/)
- analysis folder name (NAME) and reference database path (LINKPATH_DB) configured in metagenomics_analysis.sh

## Outputs

- de novo assembled contigs per sample and from co-assembly (MEGAHIT, SPAdes)
- assembly statistics reports
- filtered contigs with minimum 1000 bp length for downstream coverage-based binning

## How to apply

After the quality-control stage (run_qc.sh: FastQC reporting plus Trimmomatic/Sickle/BBMap removal of adapters, low-quality reads, sequencing artifacts, phix adapters and host contamination), feed the cleaned paired-end reads into the assembly step of the metagenomics workflow. Run single-sample and co-assembly using MEGAHIT, SPAdes and IDBA (co-assembly uses MEGAHIT and SPAdes), then generate assembly statistics. Apply the assembly filter to retain only contigs with a minimum length of 1000 bp — this removes short fragments that would degrade coverage-based binning (BBMap backmapping, MetaBAT/MaxBin) and kraken2-based bin taxonomy assignment. The assembly step is one of the sub-scripts orchestrated by metagenomics_analysis.sh, each guarded by a check_and_install function that installs missing tools; expect assembly and reference-based alignment to be the most time-consuming stages, requiring roughly 16+ cores and 100 GB RAM on a modest configuration.

## Related tools

- **MEGAHIT** (assembler for single-sample and co-assembly of quality-controlled metagenomic reads)
- **SPAdes** (assembler for single-sample and co-assembly of quality-controlled metagenomic reads)
- **IDBA** (third integrated assembler used for single-sample assembly in the workflow)
- **BBMap** (later backmaps filtered contigs to their reads to generate sorted BAM and depth files for coverage-based binning)
- **Trimmomatic** (upstream adapter, low-quality-read and host-contamination removal before assembly)
- **Sickle** (upstream low-quality read trimming before assembly)

## Examples

```
sh metagenomics_analysis.sh
```

## Evaluation signals

- Running 'sh metagenomics_analysis.sh' completes the assembly sub-script (between run_qc and run_reference_analysis) without failure, with stdout status/progress messages confirming the check_and_install-guarded assemblers were resolved or installed
- Assembly statistics are generated for each single-sample assembly and for the co-assembly before filtering
- All retained contigs have length >= 1000 bp after the assembly filter step
- Filtered contigs can be successfully backmapped to reads with BBMap to produce sorted BAM and depth files for MetaBAT/MaxBin binning

## Limitations

- Assembly is among the most computationally intensive stages: a modest configuration needs 16+ cores and 100 GB of RAM with 1 TB of diskspace (the reference databases dominate diskspace); the authors' setup used a 20-core CPU with 128 GB
- Sequencing errors and genomic repeats in microbiome data affect analysis, and metagenomic sequences may contain large amounts of unknown 'dark matter' that resists assembly or classification
- Tools must be present before running — the workflow will fail if prerequisite software (gcc/g++, java, python2.7/python3 with Bio libraries, perl Bio, R, git, metabat in PATH, checkM under /usr/local/bin) is not installed

## Evidence

- [methods] Three assemblers (Megahit,SPAdes and IDBA) are integrated in this step: "Three assemblers (Megahit,SPAdes and IDBA) are integrated in this step"
- [methods] assembly filter to min 1000 bp contigs: "After the stats, assembly filter is performed to filter contigs with minimum 1000bp length"
- [methods] preprocessing route into assembly and reference-based analysis: "removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap"
- [readme] assembly is time-consuming: "The most time consuming tasks are the ones that involve assemblies, reference based alignment."
- [readme] paired-end input naming requirement: "The sample reads must always be paired-end, demultiplexed and compressed in the\*.fastq.gz format."
- [abstract] coverage of assembly in best-practice protocol: "best-practice protocols on experimental design, sample processing, sequencing, assembly, binning, annotation and visualization"
