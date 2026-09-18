---
name: host-read-decontamination
description: Use when raw paired-end, demultiplexed Illumina FASTQ inputs (.fastq.gz)
  from host-associated microbiome samples must be cleaned of adapter, phiX, low-quality
  and host (human/mouse) contamination before assembly or profiling.
when_to_use_negative:
- Input is already a processed OTU/ASV feature table or a 16S amplicon dataset being
  handled with the mothur/dada2 amplicon workflow rather than raw metagenomic FASTQ
  pairs
- Data are long-read metagenomic reads or single-end reads, since the workflow requires
  paired-end, demultiplexed *_1.fastq.gz / *_2.fastq.gz inputs
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: FastQC
  role: Generates a comprehensive report of data quality on raw data before cleaning
  repo: https://github.com/s-andrews/FastQC/
- name: Trimmomatic
  role: Removal of adapters and low-quality reads in the run_qc.sh step
- name: Sickle
  role: Removal of adapters, low-quality reads, sequencing artifacts, phiX adapters
    and host contamination
- name: BBMap
  role: Removes host contamination and sequencing artifacts; also used for backmapping
    contigs in later binning steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:4d674192133c2d22b2c8cd10133e234624195c23b6e400e622e8de6eeed46557
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
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# Host-read decontamination

## Summary

Removes adapters, low-quality reads, sequencing artifacts, phiX and host contamination from paired-end metagenomic FASTQ reads as the quality-control stage of the review's shotgun metagenomics best-practice workflow, using Trimmomatic, Sickle and BBMap with human/mouse reference genomes.

## When to use

Use when raw paired-end, demultiplexed Illumina FASTQ inputs (*.fastq.gz) from host-associated microbiome samples must be cleaned of adapter, phiX, low-quality and host (human/mouse) contamination before assembly or profiling.

## When NOT to use

- Input is already a processed OTU/ASV feature table or a 16S amplicon dataset being handled with the mothur/dada2 amplicon workflow rather than raw metagenomic FASTQ pairs
- Data are long-read metagenomic reads or single-end reads, since the workflow requires paired-end, demultiplexed *_1.fastq.gz / *_2.fastq.gz inputs

## Inputs

- Paired-end, demultiplexed, gzip-compressed Illumina FASTQ reads named *_1.fastq.gz and *_2.fastq.gz (e.g. metaHIT gut survey ERR011 samples)
- Human reference genome (ftp://ftp.ncbi.nih.gov/genomes/H_sapiens/Assembled_chromosomes/seq/) and mouse reference genome for host removal
- Index databases (human/mouse genomes, kraken2, MEGAN, DIAMOND-indexed NR, MetaPhlAn2 mpa_v20_m200) prepared by prepare_databases.sh

## Outputs

- Cleaned paired-end FASTQ read pairs with adapters, low-quality reads, artifacts, phiX and host contamination removed
- FastQC quality report on the raw data
- stdout status/progress messages from run_qc.sh

## How to apply

First run FastQC on the raw data to generate a comprehensive data-quality report. Then execute the review's run_qc.sh quality-control script, which removes adapters, low-quality reads, sequencing artifacts, phiX adapters and host contamination using Trimmomatic, Sickle and BBMap; BBMap performs host removal by mapping reads against the pre-built human and mouse reference genomes fetched by prepare_databases.sh. Inputs must be paired-end, demultiplexed and gzip-compressed as *_1.fastq.gz and *_2.fastq.gz (e.g. the metaHIT gut survey ERR011 reads), and the reference directory is supplied via the LINKPATH_DB variable in metagenomics_analysis.sh. The rationale is that sequencing errors, artifacts and host carryover would otherwise bias assembly (MEGAHIT/SPAdes/IDBA), taxonomic profiling (kraken2, MetaPhlAn2) and binning, so cleaning precedes all downstream analysis.

## Related tools

- **FastQC** (Generates a comprehensive report of data quality on raw data before cleaning) — https://github.com/s-andrews/FastQC/
- **Trimmomatic** (Removal of adapters and low-quality reads in the run_qc.sh step)
- **Sickle** (Removal of adapters, low-quality reads, sequencing artifacts, phiX adapters and host contamination)
- **BBMap** (Removes host contamination and sequencing artifacts; also used for backmapping contigs in later binning steps)

## Examples

```
sh metagenomics_analysis.sh
```

## Evaluation signals

- Running 'sh metagenomics_analysis.sh' completes all sub-scripts (run_qc through run_bin_functional_classification) without failure
- stdout status/progress messages confirm each check_and_install-guarded tool (including Trimmomatic, Sickle, BBMap) was resolved or installed
- FastQC report generated on raw data before trimming, showing the QC stage executed
- Input FASTQ pairs conform to the required *_1.fastq.gz / *_2.fastq.gz paired, demultiplexed, gzip-compressed naming before QC starts
- Human and mouse reference genomes were downloaded and indexed under the LINKPATH_DB references directory prior to host-removal mapping

## Limitations

- Reference databases (human/mouse genomes, kraken, NR, etc.) are huge and take a long time to download and index; existing databases must be supplied via corrected paths in the scripts
- Installation of tools like MEGAN is interactive and memory-intensive (between ~3GB-16GB depending on sample size), and a modest hardware configuration is 16+ cores, 100 GB RAM and 1 TB diskspace
- The workflow was only tested on Ubuntu 18.04.2 LTS; other Linux distros should work but are unverified

## Evidence

- [methods] removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap: "removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap"
- [methods] FastQC is used to generate comprehensive report of data quality on raw data: "FastQC is used to generate comprehensive report of data quality on raw data"
- [readme] The sample reads must always be paired-end, demultiplexed and compressed in the*.fastq.gz format. Also the names of the pair must end with *_1.fastq.gz and *_2.fastq.gz.: "The sample reads must always be paired-end, demultiplexed and compressed in the\*.fastq.gz format. Also the names of the pair must end with \*\_1.fastq.gz and \*\_2.fastq.gz."
- [readme] Human and Mouse reference genome: ftp://ftp.ncbi.nih.gov/genomes/H_sapiens/Assembled_chromosomes/seq/: "Human and Mouse reference genome:
  ftp://ftp.ncbi.nih.gov/genomes/H_sapiens/Assembled_chromosomes/seq/"
