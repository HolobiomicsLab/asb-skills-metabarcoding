---
name: metagenome-contig-binning
description: Use when shotgun metagenomic assemblies (contigs filtered to a minimum
  length of 1000 bp) must be grouped into draft genome bins and assessed for quality,
  taxonomy, and function.
when_to_use_negative:
- The analysis is 16S rRNA amplicon (mothur/dada2 OTU/ASV) data — there are no metagenomic
  contigs to bin.
- The assembly still contains contigs shorter than 1000 bp; the workflow's assembly
  filter must run before binning.
- A bin set already exists and only functional annotation or taxonomy assignment is
  needed — skip straight to Prokka/kraken2 rather than re-binning.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: BBMap
  role: Backmaps contigs to their reads to generate sorted BAM and depth files used
    as coverage evidence for binning
- name: MetaBAT
  role: Coverage-based binner producing candidate genome bins from depth profiles
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- name: MaxBin
  role: Second coverage-based binner; its bins are compared/merged with MetaBAT's
- name: Binning_refiner
  role: Refines and combines the MetaBAT and MaxBin bin sets
- name: CheckM
  role: Estimates genome completeness and contamination of refined bins from marker
    genes
  repo: https://github.com/Ecogenomics/CheckM/wiki/Installation
- name: kraken2
  role: Assigns taxonomy to the refined bins
  repo: https://github.com/DerrickWood/kraken2
- name: Prokka
  role: 'Functionally annotates bins: gene prediction via Prodigal and homology search
    against CDD, PFAM and TIGRAM databases'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:68c16d82be7c74dfc72b060e1284e0e551bb40662e2cea9b1ed2410646afcdc5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBMap
  - Binning_refiner
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# metagenome-contig-binning

## Summary

A best-practice coverage-based binning step for shotgun metagenomics that groups ≥1000 bp contigs from single-sample and co-assembly into genome bins using MetaBAT and MaxBin, refines and validates them with Binning_refiner and CheckM, then assigns taxonomy and function to the bins.

## When to use

Use when shotgun metagenomic assemblies (contigs filtered to a minimum length of 1000 bp) must be grouped into draft genome bins and assessed for quality, taxonomy, and function.

## When NOT to use

- The analysis is 16S rRNA amplicon (mothur/dada2 OTU/ASV) data — there are no metagenomic contigs to bin.
- The assembly still contains contigs shorter than 1000 bp; the workflow's assembly filter must run before binning.
- A bin set already exists and only functional annotation or taxonomy assignment is needed — skip straight to Prokka/kraken2 rather than re-binning.

## Inputs

- filtered contigs from single-sample and co-assembly (MEGAHIT, SPAdes, IDBA), minimum 1000 bp
- quality-controlled paired-end reads (*_1.fastq.gz / *_2.fastq.gz, e.g. metaHIT ERR011 samples) for backmapping
- checkM marker-gene database (https://data.ace.uq.edu.au/public/CheckM_databases/)

## Outputs

- sorted BAM files and depth files from BBMap contig-to-read backmapping
- metagenome bins from MetaBAT and MaxBin, refined via Binning_refiner
- CheckM completeness and contamination estimates per bin
- kraken2 taxonomy assignments per bin
- Prokka gene predictions (Prodigal) and functional annotations against CDD, PFAM and TIGRAM databases

## How to apply

Within the grimmlab/MicrobiomeBestPracticeReview metagenomics workflow (run_coverage_bining.sh, called by metagenomics_analysis.sh), first backmap assembled contigs to their original reads with BBMap to generate sorted BAM files and per-contig depth files — coverage profiles are the signal MetaBAT and MaxBin use to bin contigs. Run MetaBAT and MaxBin in parallel on the same depth/BAM evidence to produce two candidate bin sets, then merge/refine them with Binning_refiner to reduce cross-contamination between bins. Assess every refined bin with CheckM, which estimates completeness and contamination from ubiquitous single-copy marker genes; before use, point CheckM at its marker-gene database via `checkm data setRoot PATH_TO_DOWNLOADED_DATABASE`. Assign taxonomy to the refined bins with kraken2 and functionally annotate them with Prokka, which performs gene prediction via Prodigal and homology search against CDD, PFAM and TIGRfam (TIGRAM) databases. Binning must be preceded by the workflow's assembly filter that keeps only contigs of minimum 1000 bp length.

## Related tools

- **BBMap** (Backmaps contigs to their reads to generate sorted BAM and depth files used as coverage evidence for binning)
- **MetaBAT** (Coverage-based binner producing candidate genome bins from depth profiles) — https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- **MaxBin** (Second coverage-based binner; its bins are compared/merged with MetaBAT's)
- **Binning_refiner** (Refines and combines the MetaBAT and MaxBin bin sets)
- **CheckM** (Estimates genome completeness and contamination of refined bins from marker genes) — https://github.com/Ecogenomics/CheckM/wiki/Installation
- **kraken2** (Assigns taxonomy to the refined bins) — https://github.com/DerrickWood/kraken2
- **Prokka** (Functionally annotates bins: gene prediction via Prodigal and homology search against CDD, PFAM and TIGRAM databases)

## Examples

```
sh metagenomics_analysis.sh
```

## Evaluation signals

- Input contigs passed the assembly filter with minimum 1000 bp length before binning was invoked.
- BBMap backmapping produced sorted BAM and depth files for every assembled sample before MetaBAT/MaxBin ran.
- Two independent bin sets (MetaBAT and MaxBin) were produced and merged via Binning_refiner rather than a single binner used alone.
- Each refined bin has CheckM completeness/contamination values, and `checkm data setRoot` points to a downloaded CheckM database.
- Executing `sh metagenomics_analysis.sh` completes run_coverage_bining.sh without failure, with stdout status messages confirming each check_and_install-guarded tool (metabat, checkm) was resolved or installed.

## Limitations

- CheckM v1 is unmaintained; CheckM2 (ML-based) is generally more accurate and the CheckM README suggests running both, as they are independent methodologies.
- MetaBAT must be visible in the system PATH and checkM under /usr/local/bin, or the workflow fails; checkM's database must be manually downloaded and configured.
- Binning is compute-intensive — the README recommends 16+ cores, 100 GB RAM and 1 TB disk (mostly for reference databases), tested on Ubuntu 18.04.2.

## Evidence

- [methods] After the stats, assembly filter is performed to filter contigs with minimum 1000bp length: "After the stats, assembly filter is performed to filter contigs with minimum 1000bp length"
- [readme] checkM database must be built before use: "After installation the checkM database needs to be built using https://data.ace.uq.edu.au/public/CheckM_databases/"
- [readme] MetaBAT installation prerequisite: "Install instructions can be found under https://bitbucket.org/berkeleylab/metabat/src/master/README.md. Metabat should be visible in the system PATH."
- [abstract] workflow covers binning stage: "best-practice protocols on experimental design, sample processing, sequencing, assembly, binning, annotation and visualization"
- [readme] CheckM v1 vs v2 methodology: [evidence span withheld — anchor status: external_doc]
