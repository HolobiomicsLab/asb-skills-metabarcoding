---
name: metagenomic-read-quality-control
description: Use when the input is raw paired-end, demultiplexed, gzip-compressed
  Illumina FASTQ (.fastq.gz with _1/_2 pairs, e.g. metaHIT ERR011 samples) that needs
  adapter/low-quality/phiX/host trimming before metagenomic assembly or read classification.
when_to_use_negative:
- Input is already trimmed/cleaned FASTQ or a processed feature/OTU table rather than
  raw gzipped paired-end reads
- Input is 16S rRNA amplicon data — the paper handles that in a separate mothur/dada2
  amplicon workflow (amplicon_analysis.sh)
- Input reads are not paired-end, not demultiplexed, or not compressed as *_1.fastq.gz
  / *_2.fastq.gz as the workflow requires
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: FastQC
  role: generates comprehensive report of data quality on the raw FASTQ data
  repo: https://github.com/s-andrews/FastQC/
- name: Trimmomatic
  role: removal of adapters and low-quality reads
- name: Sickle
  role: removal of low-quality reads
- name: BBMap
  role: removal of sequencing artifacts, phiX adapters and host contamination using
    human/mouse reference genomes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:60df8ed3f9cc714da80f6d31374aadb46871d512b2aed4a17d21471167cfb303
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/s-andrews/FastQC
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.10526315789473684
n_claims: 19
---

# metagenomic-read-quality-control

## Summary

Quality-control step of the paper's shotgun metagenomics workflow (run_qc.sh): FastQC reporting on raw reads followed by removal of adapters, low-quality reads, sequencing artifacts, phiX and host contamination with Trimmomatic, Sickle and BBMap, producing cleaned paired-end FASTQ for assembly and reference-based routes.

## When to use

Use when the input is raw paired-end, demultiplexed, gzip-compressed Illumina FASTQ (*.fastq.gz with *_1/*_2 pairs, e.g. metaHIT ERR011 samples) that needs adapter/low-quality/phiX/host trimming before metagenomic assembly or read classification.

## When NOT to use

- Input is already trimmed/cleaned FASTQ or a processed feature/OTU table rather than raw gzipped paired-end reads
- Input is 16S rRNA amplicon data — the paper handles that in a separate mothur/dada2 amplicon workflow (amplicon_analysis.sh)
- Input reads are not paired-end, not demultiplexed, or not compressed as *_1.fastq.gz / *_2.fastq.gz as the workflow requires

## Inputs

- Raw paired-end, demultiplexed, gzip-compressed Illumina reads named *_1.fastq.gz and *_2.fastq.gz (e.g. metaHIT gut survey, ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/)
- Human and mouse reference genomes (NCBI, under LINKPATH_DB references directory) for host-contamination removal

## Outputs

- FastQC data-quality report on the raw reads
- Quality- and adapter-trimmed, host-decontaminated paired FASTQ reads passed to assembly (MEGAHIT, SPAdes, IDBA) and reference-based analysis (kraken2, MetaPhlAn2, DIAMOND, HUMAnN2)
- stdout status/progress messages from check_and_install-guarded tools

## How to apply

Run run_qc.sh (invoked via sh metagenomics_analysis.sh) on the local copy of raw data declared through SRC_RAWDATA. First generate a comprehensive quality report on the raw data with FastQC (https://github.com/s-andrews/FastQC/). Then remove adapters, low-quality reads, sequencing artifacts, phiX adapters and host (human/mouse) contamination using Trimmomatic, Sickle and BBMap, where BBMap uses the Homo sapiens and Mus musculus reference genomes prepared by prepare_databases.sh under the references directory (LINKPATH_DB). Each required tool is resolved by a per-sub-script check_and_install function that installs it if missing. The cleaned reads feed two downstream routes: single-sample and co-assembly with MEGAHIT/SPAdes/IDBA, and reference-based profiling with kraken2, MetaPhlAn2 and DIAMOND/MEGAN/HUMAnN2.

## Related tools

- **FastQC** (generates comprehensive report of data quality on the raw FASTQ data) — https://github.com/s-andrews/FastQC/
- **Trimmomatic** (removal of adapters and low-quality reads)
- **Sickle** (removal of low-quality reads)
- **BBMap** (removal of sequencing artifacts, phiX adapters and host contamination using human/mouse reference genomes)

## Examples

```
sh metagenomics_analysis.sh
```

## Evaluation signals

- FastQC report produced on the raw reads before trimming
- Quality-control sub-script run_qc.sh completes without failure when sh metagenomics_analysis.sh is executed
- stdout status/progress messages confirm each check_and_install-guarded tool (FastQC, Trimmomatic, Sickle, BBMap) was resolved or installed
- Trimmed reads are still paired-end, demultiplexed and gzip-compressed, retaining the *_1.fastq.gz / *_2.fastq.gz naming convention
- Downstream steps (MEGAHIT/SPAdes/IDBA assembly, kraken2/MetaPhlAn2 profiling) can start on the cleaned reads without format errors

## Limitations

- Host-decontamination requires pre-downloaded human and mouse reference genomes from NCBI, which take long to download and index in prepare_databases.sh
- Workflow requires external software pre-installed (gcc/g++, java, python2.7/python3 with Bio and other libraries, perl Bio, R, git); without these the workflow will fail to run
- Heavy hardware footprint: assemblies and reference-based alignment are the most time-consuming tasks; a modest configuration is 16+ cores, 100 GB RAM and 1 TB diskspace (the authors' configuration: 20 cores, 128 GB)

## Evidence

- [methods] FastQC data-quality reporting: "[FastQC](https://github.com/s-andrews/FastQC/) is used to generate comprehensive report of data quality on raw data"
- [methods] trimming tools and targets: "removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap"
- [readme] input read format requirement: "The sample reads must always be paired-end, demultiplexed and compressed in the\*.fastq.gz format. Also the names of the pair must end with \*\_1.fastq.gz and \*\_2.fastq.gz."
- [readme] check_and_install guard: "The "check_and_install" checks for the tools required to run the respective script and installs them if they are missing."
