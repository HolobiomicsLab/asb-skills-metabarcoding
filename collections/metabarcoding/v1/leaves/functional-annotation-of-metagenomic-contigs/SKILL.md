---
name: functional-annotation-of-metagenomic-contigs
description: Use when working from assembled, length-filtered metagenomic contigs
  (>=1000 bp) or refined MAGs and the goal is gene prediction, homology-based functional
  annotation, or taxonomic/functional profiling of bins rather than raw-read QC.
when_to_use_negative:
- Input is 16S rRNA amplicon data — the review routes amplicon analysis through the
  separate mothur/dada2 amplicon workflow (amplicon_analysis.sh), not the metagenomics
  binning/annotation pipeline.
- Input is raw, unfiltered FASTQ or reads shorter than the QC/assembly stage — annotation
  requires assembly output with contigs of minimum 1000 bp.
- The goal is only OTU/ASV abundance profiling (e.g. OTU clustering at 97% identity
  or ASV denoising) rather than function of assembled contigs or bins.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: DIAMOND
  role: Performs blastx alignment of processed reads against the NR database; output
    is meganized for MEGAN6 functional/taxonomic classification
- name: MEGAN
  role: Consumes meganized DIAMOND output for visualization and functional profiling
    of classified reads
- name: HUMAnN2
  role: Reference-based functional profiling of metagenomic reads using the HUMAnN2
    database
- name: kraken2
  role: Classifies output reads and assigns taxonomy to refined bins
  repo: github.com/DerrickWood/kraken2
- name: CheckM
  role: Refines metagenomic bins before their functional annotation with Prokka
  repo: github.com/Ecogenomics/CheckM
- name: Prokka
  role: 'Functionally annotates refined bins: gene prediction via Prodigal and homology
    search against CDD, PFAM and TIGRAM databases'
- name: BBMap
  role: Backmaps contigs to their reads to generate sorted BAM and depth files used
    for coverage-based binning preceding annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:e109604b2dd26a85ae4c1d87280b06bda87d7ee2d27365bc6886ee02782f3522
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DerrickWood/kraken2
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.36
n_claims: 25
---

# functional-annotation-of-metagenomic-contigs

## Summary

Assigns function to assembled metagenomic contigs and refined bins by predicting open reading frames and searching them against curated protein/homology databases (NR, CDD, PFAM, TIGRAM) and read-classification outputs (kraken2, DIAMOND/Megan, HUMAnN2). This step turns the binned assembly of the review's shotgun metagenomics workflow into functional profiles, which the review lists among its core best-practice stages (assembly, binning, annotation, visualization).

## When to use

Use when working from assembled, length-filtered metagenomic contigs (>=1000 bp) or refined MAGs and the goal is gene prediction, homology-based functional annotation, or taxonomic/functional profiling of bins rather than raw-read QC.

## When NOT to use

- Input is 16S rRNA amplicon data — the review routes amplicon analysis through the separate mothur/dada2 amplicon workflow (amplicon_analysis.sh), not the metagenomics binning/annotation pipeline.
- Input is raw, unfiltered FASTQ or reads shorter than the QC/assembly stage — annotation requires assembly output with contigs of minimum 1000 bp.
- The goal is only OTU/ASV abundance profiling (e.g. OTU clustering at 97% identity or ASV denoising) rather than function of assembled contigs or bins.

## Inputs

- Processed paired-end metagenomic FASTQ reads (*_1.fastq.gz / *_2.fastq.gz) after QC (FastQC/Trimmomatic/Sickle/BBMap)
- Filtered contigs of minimum 1000 bp from single-sample or co-assembly (MEGAHIT/SPAdes/IDBA)
- Refined metagenomic bins (MetaBAT/MaxBin via Binning_refiner, refined with CheckM)
- Prepared reference databases: NR (DIAMOND-indexed), MEGAN database, MetaPhlAn2 mpa_v20_m200, kraken2 index, checkM database, HUMAnN2 database

## Outputs

- Meganized DIAMOND blastx outputs against NR for MEGAN6 visualization and classification
- HUMAnN2 functional profiles of the reads
- kraken2 taxonomic classifications of reads and bins
- Prokka gene predictions (Prodigal) and functional annotations per bin (homology search against CDD, PFAM, TIGRAM)
- Sorted BAM and depth files from BBMap backmapping used by binners

## How to apply

First run reference-based profiling on processed reads (run_reference_analysis.sh): DIAMOND blastx against the indexed NR database, meganize the output for MEGAN6 visualization, and compute functional profiles with HUMAnN2; kraken2 and MetaPhlAn2 classify reads for taxonomy. Then, after coverage-based binning (BBMap backmapping to sorted BAM/depth files, MetaBAT, MaxBin, Binning_refiner, CheckM refinement), assign bin taxonomy with kraken2 and functionally annotate the bins with Prokka, which performs gene prediction via Prodigal and homology search against the CDD, PFAM and TIGRAM databases. All reference databases (kraken2, MEGAN, NR indexed with DIAMOND, MetaPhlAn2 mpa_v20_m200 built with bowtie2, checkM, HUMAnN2) must be installed and indexed beforehand via prepare_databases.sh, with the resulting path plugged into LINKPATH_DB. Correct application means each check_and_install-guarded tool resolves or installs, sub-scripts run without failure, and outputs (meganized DIAMOND hits, HUMAnN2 profiles, Prokka GFF/annotation per bin) are produced for every bin.

## Related tools

- **DIAMOND** (Performs blastx alignment of processed reads against the NR database; output is meganized for MEGAN6 functional/taxonomic classification)
- **MEGAN** (Consumes meganized DIAMOND output for visualization and functional profiling of classified reads)
- **HUMAnN2** (Reference-based functional profiling of metagenomic reads using the HUMAnN2 database)
- **kraken2** (Classifies output reads and assigns taxonomy to refined bins) — github.com/DerrickWood/kraken2
- **CheckM** (Refines metagenomic bins before their functional annotation with Prokka) — github.com/Ecogenomics/CheckM
- **Prokka** (Functionally annotates refined bins: gene prediction via Prodigal and homology search against CDD, PFAM and TIGRAM databases)
- **BBMap** (Backmaps contigs to their reads to generate sorted BAM and depth files used for coverage-based binning preceding annotation)

## Examples

```
sh metagenomics_analysis.sh
```

## Evaluation signals

- Executing 'sh metagenomics_analysis.sh' completes all sub-scripts (run_qc through run_bin_functional_classification) without failure.
- stdout status/progress messages confirm each check_and_install-guarded tool was resolved or installed.
- DIAMOND blastx output against NR is successfully meganized and importable into MEGAN6.
- Every refined bin has kraken2 taxonomic assignment and Prokka annotation output (gene predictions via Prodigal with CDD/PFAM/TIGRAM hits).
- Filtered assembly contains only contigs of minimum 1000 bp length prior to binning and annotation.

## Limitations

- Reference databases are huge and take a long time to download and index; a modest configuration of 16+ cores, 100 GB RAM and 1 TB disk space is recommended (authors tested with 20 cores / 128 GB RAM).
- MEGAN installation is interactive and requires the user to input Y/N and memory options (~3GB-16GB depending on sample size).
- Some software (metabat, checkM with its manually downloaded database at /usr/local/bin) must be installed by the user beforehand or the workflow will fail to run.

## Evidence

- [methods] Read classification and functional profiling tools: "different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads"
- [methods] DIAMOND/Megan route: "Diamond blastx is performed against NR database and meganized using megan databases"
- [readme] Database prerequisite: "The databases form the core of the workflows. Unfortunately, the databases are huge and  take a long time to download and index."
- [readme] Workflow execution feedback: "There are messages on the stdout showing the status and progress of the analysis."
