---
name: shotgun-metagenomics-annotation-tool-selection
description: Use when processing shotgun metagenomic reads (e.g. paired-end FASTQ
  from a gut-survey like metaHIT ERR011) and reaching the classification/annotation
  phase, needing to choose among kraken2, MetaPhlAn2, DIAMOND, MEGAN, HUMAnN2 and
  Prokka.
when_to_use_negative:
- Input is 16S rRNA amplicon data — use the mothur (97% OTU clustering) or DADA2 (ASV
  denoising) amplicon workflow instead of the metagenomic annotation pipeline
- Reads are raw and have not passed the quality-control step (FastQC plus Trimmomatic/Sickle/BBMap
  for adapters, low quality reads, PhiX and host contamination)
- Only binning completeness/contamination estimation is needed — that is CheckM's
  role in this workflow, not the annotation tool set
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: kraken2
  role: State-of-the-art classification of output metagenomic reads
  repo: github.com/DerrickWood/kraken2
- name: MetaPhlAn2
  role: Marker-based taxonomic profiling of metagenomic reads, using the mpa_v20_m200
    database built with bowtie2
- name: DIAMOND
  role: Performs blastx alignment of translated reads/genes against the NR database
- name: MEGAN
  role: Consumes meganized DIAMOND blastx results for taxonomic and functional interpretation;
    requires interactive installation with 3-16 GB memory options
- name: HUMAnN2
  role: Functional profiling of metagenomic reads; its databases are installed via
    humann2_databases
- name: Prokka
  role: Annotation of assembled contigs
- name: Prodigal
  role: Gene prediction used together with Prokka
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:3930771334c7e92a976a8915d39c0190d132d753200fb262f04dfb0b596fd7c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBMap
  - CheckM
  - DADA2
  - DIAMOND
  - FastQC
  - HUMAnN2
  - IDBA
  - MEGAHIT
  - MEGAN
  - MaxBin
  - MetaBAT
  - MetaPhlAn2
  - Prodigal
  - Prokka
  - SPAdes
  - Sickle
  - Trimmomatic
  - kraken2
  - mothur
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.64
n_claims: 25
---

# shotgun-metagenomics-annotation-tool-selection

## Summary

Selects and deploys the review's best-practice tools for functional annotation and read classification in the shotgun metagenomics workflow of Bharti & Grimm (doi:10.1093/bib/bbz155): DIAMOND blastx against the NR database, MEGAN, HUMAnN2, kraken2, MetaPhlAn2, and Prokka with Prodigal for gene prediction and annotation.

## When to use

Use when processing shotgun metagenomic reads (e.g. paired-end FASTQ from a gut-survey like metaHIT ERR011) and reaching the classification/annotation phase, needing to choose among kraken2, MetaPhlAn2, DIAMOND, MEGAN, HUMAnN2 and Prokka.

## When NOT to use

- Input is 16S rRNA amplicon data — use the mothur (97% OTU clustering) or DADA2 (ASV denoising) amplicon workflow instead of the metagenomic annotation pipeline
- Reads are raw and have not passed the quality-control step (FastQC plus Trimmomatic/Sickle/BBMap for adapters, low quality reads, PhiX and host contamination)
- Only binning completeness/contamination estimation is needed — that is CheckM's role in this workflow, not the annotation tool set

## Inputs

- Paired-end, demultiplexed, gzipped Illumina FASTQ files named *_1.fastq.gz and *_2.fastq.gz (e.g. metaHIT gut survey reads from ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/)
- Assembled contigs of minimum 1000 bp length from the assembly step (MEGAHIT/SPAdes/IDBA output)
- Reference databases: NR database (nr.gz), kraken2-indexed Kraken database, MEGAN database, MetaPhlAn2 mpa_v20_m200, checkM database, HUMAnN2 databases

## Outputs

- Taxonomic classification of reads (kraken2 and MetaPhlAn2 reports)
- DIAMOND blastx matches against NR, meganized for inspection in MEGAN
- Functional profiles/abundance estimates from HUMAnN2
- Gene predictions and functional annotations of contigs from Prokka (with Prodigal)

## How to apply

First prepare the required reference databases with prepare_databases.sh: Human and Mouse genomes for host removal, a kraken2-indexed Kraken database, the MEGAN database from ab.inf.uni-tuebingen.de, the NR database indexed with DIAMOND, the MetaPhlAn2 database (mpa_v20_m200) built with bowtie2, the checkM database, and the HUMAnN2 databases. After quality control (FastQC plus Trimmomatic/Sickle/BBMap) and assembly (MEGAHIT/SPAdes/IDBA) with contigs filtered to a minimum of 1000 bp, classify output reads with kraken2 and MetaPhlAn2; run DIAMOND blastx against the NR database and 'meganize' the results with MEGAN databases for taxonomic/functional interpretation; use HUMAnN2 for functional profiling. For gene prediction and annotation of assembled contigs, use Prokka with Prodigal. All steps are orchestrated by the metagenomics_analysis.sh bash script, which uses check_and_install functions to verify each tool is present before running.

## Related tools

- **kraken2** (State-of-the-art classification of output metagenomic reads) — github.com/DerrickWood/kraken2
- **MetaPhlAn2** (Marker-based taxonomic profiling of metagenomic reads, using the mpa_v20_m200 database built with bowtie2)
- **DIAMOND** (Performs blastx alignment of translated reads/genes against the NR database)
- **MEGAN** (Consumes meganized DIAMOND blastx results for taxonomic and functional interpretation; requires interactive installation with 3-16 GB memory options)
- **HUMAnN2** (Functional profiling of metagenomic reads; its databases are installed via humann2_databases)
- **Prokka** (Annotation of assembled contigs)
- **Prodigal** (Gene prediction used together with Prokka)

## Examples

```
sh prepare_databases.sh && sh metagenomics_analysis.sh
```

## Evaluation signals

- Every reference database (NR, Kraken, MEGAN, MetaPhlAn mpa_v20_m200, checkM, HUMAnN2) is present and correctly indexed (diamond-indexed NR, bowtie2-built MetaPhlAn, kraken2-indexed Kraken DB) before the workflow starts
- check_and_install finds every annotation/classification tool on the system PATH (including metabat and checkM under /usr/local/bin) so metagenomics_analysis.sh runs without failure
- All input samples are paired-end, demultiplexed *_1.fastq.gz / *_2.fastq.gz; reads failing this naming convention abort the workflow
- Contigs entering annotation are >= 1000 bp, matching the assembly-filter threshold
- MEGAN's interactive install completed with the recommended default memory options (3-16 GB depending on sample size)

## Limitations

- Reference databases are huge and take a long time to download and index, dominating disk space (workflow assumes ~1 TB disk, 100 GB RAM, 16+ cores; tested on 20 cores/128 GB)
- MEGAN installation is interactive (requires Y/N and memory input between ~3GB and 16GB); non-interactive environments must anticipate this
- The workflows were only tested on Ubuntu 18.04.2 LTS with gcc/g++, java, python2.7/python3, perl (Bio), and R prerequisites satisfied

## Evidence

- [methods] different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads: "different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads"
- [methods] Diamond blastx is performed against NR database and meganized using megan databases: "Diamond blastx is performed against NR database and meganized using megan databases"
- [readme] Second workflow for metagenomics, using a variety of tools openly available which have been stitched together to form a usable pipeline.: "Second workflow for metagenomics, using a variety of tools openly available which have been stitched together to form a usable pipeline."
- [readme] The databases form the core of the workflows. Unfortunately, the databases are huge and take a long time to download and index.: "The databases form the core of the workflows. Unfortunately, the databases are huge and take a long time to download and index."
- [methods] After the stats, assembly filter is performed to filter contigs with minimum 1000bp length: "After the stats, assembly filter is performed to filter contigs with minimum 1000bp length"
