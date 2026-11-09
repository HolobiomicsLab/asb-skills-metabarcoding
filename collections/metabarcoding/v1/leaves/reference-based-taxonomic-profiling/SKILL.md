---
name: reference-based-taxonomic-profiling
description: Use when quality-filtered paired-end FASTQ reads (e.g. metaHIT gut survey
  samples) need taxonomic classification against reference databases after QC and
  before or alongside assembly-based binning.
when_to_use_negative:
- Input reads have not passed quality control (adapters, low-quality reads, phix and
  host contamination still present)
- The goal is bin reconstruction or genome-resolved analysis rather than read-level
  taxonomic classification (use the coverage-based binning route with MetaBAT/MaxBin
  instead)
- Data are 16S rRNA amplicon sequences rather than shotgun metagenomic reads (use
  the mothur/dada2 amplicon workflow)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: kraken2
  role: k-mer-based taxonomic classification of quality-filtered reads against an
    indexed kraken database
  repo: http://github.com/DerrickWood/kraken2/archive/v2.0.8-beta.tar.gz
- name: MetaPhlAn2
  role: marker-gene-based taxonomic profiling using the mpa_v20_m200 database built
    with bowtie2
  repo: https://bitbucket.org/biobakery/metaphlan2/downloads/mpa_v20_m200.tar
- name: DIAMOND
  role: blastx alignment of reads against the DIAMOND-indexed NR database
- name: MEGAN
  role: visualization of meganized DIAMOND output for taxonomic and functional inspection
  repo: http://ab.inf.uni-tuebingen.de/data/software/megan6/download/
- name: HUMAnN2
  role: functional profiling of the classified reads using humann2 databases
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:b143d2c2799e172c7cfb3c441073e555b098cc4eb3f0e0344b7b37e285cc3582
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# Reference-based taxonomic profiling

## Summary

Classifies quality-controlled shotgun metagenomic reads against curated reference databases (kraken2, MetaPhlAn2, DIAMOND/NR, MEGAN) as part of the best-practice metagenomics workflow, and extends to functional profiling with HUMAnN2.

## When to use

Use when quality-filtered paired-end FASTQ reads (e.g. metaHIT gut survey samples) need taxonomic classification against reference databases after QC and before or alongside assembly-based binning.

## When NOT to use

- Input reads have not passed quality control (adapters, low-quality reads, phix and host contamination still present)
- The goal is bin reconstruction or genome-resolved analysis rather than read-level taxonomic classification (use the coverage-based binning route with MetaBAT/MaxBin instead)
- Data are 16S rRNA amplicon sequences rather than shotgun metagenomic reads (use the mothur/dada2 amplicon workflow)

## Inputs

- Quality-filtered, demultiplexed, gzip-compressed paired-end Illumina FASTQ reads (*_1.fastq.gz and *_2.fastq.gz)
- Prepared reference databases (kraken2, MetaPhlAn2 mpa_v20_m200, DIAMOND-indexed NR, MEGAN database)
- LINKPATH_DB path to the references directory

## Outputs

- Taxonomic classification of reads (kraken2 and MetaPhlAn2 reports)
- Meganized DIAMOND blastx output against NR for MEGAN6 visualization
- HUMAnN2 functional profiling of metagenomic reads

## How to apply

After run_qc.sh has removed adapters, low-quality reads, sequencing artifacts, phix and host contamination, run reference-based profiling (run_reference_analysis.sh) with kraken2 and MetaPhlAn2 for k-mer/marker-based read classification, and DIAMOND blastx against the NR database, meganizing the output for MEGAN6 visualization. The required databases are prepared by prepare_databases.sh: the kraken2 database (v2.0.8-beta) indexed with kraken2, MetaPhlAn2's mpa_v20_m200 built with bowtie2, and the NR database (ftp://ftp.ncbi.nih.gov/blast/db/FASTA/nr.gz) indexed with DIAMOND; paths are supplied via LINKPATH_DB. Functional profiling of the classified reads is then performed with HUMAnN2, whose databases are downloaded via humann2_databases. Correct application is judged by all sub-scripts completing via sh metagenomics_analysis.sh without failure, with each check_and_install-guarded tool resolved.

## Related tools

- **kraken2** (k-mer-based taxonomic classification of quality-filtered reads against an indexed kraken database) — http://github.com/DerrickWood/kraken2/archive/v2.0.8-beta.tar.gz
- **MetaPhlAn2** (marker-gene-based taxonomic profiling using the mpa_v20_m200 database built with bowtie2) — https://bitbucket.org/biobakery/metaphlan2/downloads/mpa_v20_m200.tar
- **DIAMOND** (blastx alignment of reads against the DIAMOND-indexed NR database)
- **MEGAN** (visualization of meganized DIAMOND output for taxonomic and functional inspection) — http://ab.inf.uni-tuebingen.de/data/software/megan6/download/
- **HUMAnN2** (functional profiling of the classified reads using humann2 databases)

## Examples

```
sh metagenomics_analysis.sh  (with LINKPATH_DB=/xxx/.../references/ set; internally runs run_reference_analysis.sh with kraken2, MetaPhlAn2, DIAMOND blastx against NR, and HUMAnN2)
```

## Evaluation signals

- sh metagenomics_analysis.sh completes all sub-scripts (run_qc through run_bin_functional_classification) without failure
- stdout status/progress messages confirm each check_and_install-guarded tool (kraken2, MetaPhlAn2, DIAMOND, MEGAN, HUMAnN2) was resolved or installed
- All required reference databases (kraken2-indexed, MetaPhlAn2 mpa_v20_m200 bowtie2-built, DIAMOND-indexed NR, MEGAN) exist under the LINKPATH_DB references directory before profiling runs
- DIAMOND blastx output is successfully meganized and opens in MEGAN6 for visualization

## Limitations

- The databases are huge and take a long time to download and index; if they already exist the scripts must be manually modified with the correct paths
- MEGAN installation is interactive, requiring Y/N and memory inputs (~3GB-16GB depending on sample size); default options are recommended
- Raw inputs must be paired-end, demultiplexed and compressed as *_1.fastq.gz and *_2.fastq.gz, otherwise the workflow fails

## Evidence

- [methods] reference-based profiling route: "different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads"
- [methods] DIAMOND blastx against NR: "Diamond blastx is performed against NR database and meganized using megan databases"
- [readme] input read format requirements: "The sample reads must always be paired-end, demultiplexed and compressed in the\*.fastq.gz format. Also the names of the pair must end with \*_1.fastq.gz and \*_2.fastq.gz."
- [readme] database preparation: "The databases form the core of the workflows. Unfortunately, the databases are huge and  take a long time to download and index."
