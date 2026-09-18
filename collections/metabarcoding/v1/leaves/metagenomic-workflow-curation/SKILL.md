---
name: metagenomic-workflow-curation
description: Use when a microbiome analysis plan or pipeline audit must name the recommended
  tool per metagenomic or amplicon phase (quality control, assembly, binning, classification,
  annotation) as catalogued in the Bharti & Grimm highest-ranked-practice review.
when_to_use_negative:
- The goal is to execute the pipelines on raw paired-end *.fastq.gz reads rather than
  curate the tool map - run amplicon_analysis.sh / metagenomics_analysis.sh directly
  instead
- Input is already a curated tool map (best_practice_tools_map.csv exists) and the
  task is downstream statistical or visualization analysis (e.g., phyloseq), not tool
  selection
- 'The data are non-microbiome libraries (e.g., ChIP-Seq, RNA-Seq, BS-Seq): FastQC
  itself is generic, but the review''s phase-to-tool catalogue covers only 16S rRNA
  and shotgun/long-read metagenomics'
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: FastQC
  role: 'Quality-control phase: generates the comprehensive report of data quality
    on raw reads (run_qc.sh)'
  repo: https://github.com/s-andrews/FastQC/
- name: Trimmomatic
  role: 'Quality-control phase: removal of adapters, low quality reads, sequencing
    artifacts, phix adapters and host contamination'
- name: Sickle
  role: 'Quality-control phase: quality trimming alongside Trimmomatic and BBMap'
- name: BBMap
  role: 'Quality-control phase: host-contamination and artifact removal alongside
    Trimmomatic and Sickle'
- name: MEGAHIT
  role: 'Assembly phase: one of the three integrated metagenomic assemblers'
- name: SPAdes
  role: 'Assembly phase: one of the three integrated metagenomic assemblers'
- name: IDBA
  role: 'Assembly phase: one of the three integrated metagenomic assemblers'
- name: MetaBAT
  role: 'Binning phase: coverage-based binning of assembled contigs'
  repo: bitbucket.org/berkeleylab/metabat
- name: MaxBin
  role: 'Binning phase: coverage-based binning of assembled contigs'
- name: Binning_refiner
  role: 'Bin-refinement phase: refines bins prior to quality assessment'
- name: CheckM
  role: 'Bin-refinement phase: marker-gene-based estimates of genome completeness
    and contamination'
  repo: github.com/Ecogenomics/CheckM
- name: kraken2
  role: 'Read-classification phase: state-of-the-art taxonomic classification of output
    reads'
  repo: github.com/DerrickWood/kraken2
- name: MetaPhlAn2
  role: 'Read-classification phase: marker-based taxonomic profiling of output reads
    (mpa_v20_m200 database built with bowtie2)'
- name: DIAMOND
  role: 'Read-classification phase: blastx against the NR database, output meganized
    for MEGAN'
- name: MEGAN
  role: 'Read-classification phase: assignment of meganized DIAMOND blastx hits'
- name: HUMAnN2
  role: 'Read-classification/functional phase: profiling of output reads alongside
    Kraken2, MetaPhlAn2, DIAMOND and MEGAN'
- name: Prokka
  role: 'Annotation phase: functional annotation of assembled sequences'
- name: Prodigal
  role: 'Annotation phase: gene prediction underlying Prokka annotation'
- name: mothur
  role: 'Amplicon phase: OTU clustering workflow at the commonly used 97% identity
    threshold'
- name: DADA2
  role: 'Amplicon phase: denoising-based ASV workflow'
- name: phyloseq
  role: 'Amplicon phase: standard visualization of processed OTU/ASV data'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:a0b5a18ec277e95fa01f97b2f54c74b72f899ba51b50fb7adf594d74dca02ba8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/s-andrews/FastQC
  tools:
  - BBMap
  - Binning_refiner
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
  - phyloseq
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# metagenomic-workflow-curation

## Summary

Curates the Bharti & Grimm (2019) best-practice microbiome workflows (doi:10.1093/bib/bbz155) into a phase-by-phase tool-selection map (best_practice_tools_map.csv), so future agents know which tool (e.g., Trimmomatic/Sickle/BBMap for metagenomic quality control) to apply at each 16S rRNA or shotgun-metagenomics step.

## When to use

Use when a microbiome analysis plan or pipeline audit must name the recommended tool per metagenomic or amplicon phase (quality control, assembly, binning, classification, annotation) as catalogued in the Bharti & Grimm best-practice review.

## When NOT to use

- The goal is to execute the pipelines on raw paired-end *.fastq.gz reads rather than curate the tool map - run amplicon_analysis.sh / metagenomics_analysis.sh directly instead
- Input is already a curated tool map (best_practice_tools_map.csv exists) and the task is downstream statistical or visualization analysis (e.g., phyloseq), not tool selection
- The data are non-microbiome libraries (e.g., ChIP-Seq, RNA-Seq, BS-Seq): FastQC itself is generic, but the review's phase-to-tool catalogue covers only 16S rRNA and shotgun/long-read metagenomics

## Inputs

- Methods text of Bharti & Grimm (2019), 'Current challenges and best-practice protocols for microbiome analysis', doi:10.1093/bib/bbz155
- README of the grimmlab/MicrobiomeBestPracticeReview repository (driver scripts amplicon_analysis.sh and metagenomics_analysis.sh)
- Optional validation input: paired-end, demultiplexed *.fastq.gz metagenomic reads (e.g., metaHIT gut survey samples from ftp.sra.ebi.ac.uk/vol1/fastq/ERR011/)

## Outputs

- best_practice_tools_map.csv with one row per analysis phase and columns: phase, data type (metagenomic/amplicon), best-practice tool(s), tool role
- Coverage checklist confirming all nine metagenomic workflow steps plus the mothur and DADA2 amplicon workflows are represented

## How to apply

Parse the methods of the review (doi:10.1093/bib/bbz155) and the grimmlab/MicrobiomeBestPracticeReview README to enumerate every named analysis phase, then record the explicitly named tools per phase: FastQC for the comprehensive report of data quality on raw reads; Trimmomatic, Sickle and BBMap for removal of adapters, low-quality reads, sequencing artifacts, phix adapters and host contamination; MEGAHIT, SPAdes and IDBA as the three integrated assemblers (with post-assembly filtering of contigs below 1000bp); MetaBAT and MaxBin for coverage-based binning, refined with Binning_refiner and assessed by CheckM. For read classification, record Kraken2, MetaPhlAn2, DIAMOND (blastx against the NR database, meganized via MEGAN databases), MEGAN and HUMAnN2, and record Prokka with Prodigal gene prediction for annotation. For the amplicon branch, preserve the review's OTU-vs-ASV distinction: mothur-style OTU clustering at the commonly used 97% identity threshold (with SILVA nr_v132, Greengenes Gg_13_8_99 or RDP Trainset16_022016 reference sets) versus DADA2's denoising-based ASV approach. Assemble one CSV row per phase with columns phase, data type (metagenomic/amplicon), best-practice tool(s) and the tool's role, writing the result as best_practice_tools_map.csv. Validate that every metagenomic step of metagenomics_analysis.sh (steps 1-9, from run_qc.sh quality control through annotation) plus both the mothur and DADA2 amplicon workflows are represented, and that no tool name appears that is not verbatim in the source (the review pins no tool versions).

## Related tools

- **FastQC** (Quality-control phase: generates the comprehensive report of data quality on raw reads (run_qc.sh)) — https://github.com/s-andrews/FastQC/
- **Trimmomatic** (Quality-control phase: removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination)
- **Sickle** (Quality-control phase: quality trimming alongside Trimmomatic and BBMap)
- **BBMap** (Quality-control phase: host-contamination and artifact removal alongside Trimmomatic and Sickle)
- **MEGAHIT** (Assembly phase: one of the three integrated metagenomic assemblers)
- **SPAdes** (Assembly phase: one of the three integrated metagenomic assemblers)
- **IDBA** (Assembly phase: one of the three integrated metagenomic assemblers)
- **MetaBAT** (Binning phase: coverage-based binning of assembled contigs) — bitbucket.org/berkeleylab/metabat
- **MaxBin** (Binning phase: coverage-based binning of assembled contigs)
- **Binning_refiner** (Bin-refinement phase: refines bins prior to quality assessment)
- **CheckM** (Bin-refinement phase: marker-gene-based estimates of genome completeness and contamination) — github.com/Ecogenomics/CheckM
- **kraken2** (Read-classification phase: state-of-the-art taxonomic classification of output reads) — github.com/DerrickWood/kraken2
- **MetaPhlAn2** (Read-classification phase: marker-based taxonomic profiling of output reads (mpa_v20_m200 database built with bowtie2))
- **DIAMOND** (Read-classification phase: blastx against the NR database, output meganized for MEGAN)
- **MEGAN** (Read-classification phase: assignment of meganized DIAMOND blastx hits)
- **HUMAnN2** (Read-classification/functional phase: profiling of output reads alongside Kraken2, MetaPhlAn2, DIAMOND and MEGAN)
- **Prokka** (Annotation phase: functional annotation of assembled sequences)
- **Prodigal** (Annotation phase: gene prediction underlying Prokka annotation)
- **mothur** (Amplicon phase: OTU clustering workflow at the commonly used 97% identity threshold)
- **DADA2** (Amplicon phase: denoising-based ASV workflow)
- **phyloseq** (Amplicon phase: standard visualization of processed OTU/ASV data)

## Examples

```
sh metagenomics_analysis.sh
```

## Evaluation signals

- best_practice_tools_map.csv has exactly one row per named phase (quality control, assembly, binning, taxonomic classification, functional annotation, comparative/statistical analysis, plus amplicon OTU and ASV rows) with no missing or duplicated phase/data-type combinations
- The quality-control row names Trimmomatic, Sickle and BBMap for adapter, low-quality-read, sequencing-artifact, phix-adapter and host-contamination removal, with FastQC as the raw-data quality report
- The assembly row lists all three assemblers (MEGAHIT, SPAdes, IDBA) and the catalogue records the post-assembly filter of contigs with minimum 1000bp length
- Amplicon rows separate OTU clustering (mothur, commonly 97% identity) from ASV denoising (DADA2) and name the SILVA nr_v132 / Greengenes Gg_13_8_99 / RDP Trainset16_022016 reference choices
- Every tool name in the catalogue occurs verbatim in the review methods or README - no versions, repositories or parameters are invented, since the review names no tool versions

## Limitations

- The review names only 'optimally tested tools' without versions, so the catalogue cannot pin releases, and the grimmlab repository README provides no changelog to trace protocol drift
- The metagenomic workflow was tested on Ubuntu 18.04.2 with 16+ cores and ~100 GB RAM and expects paired-end, demultiplexed *.fastq.gz input (metaHIT ERR011 example), so tool assignments may not transfer to single-end, long-read or other-habitat data
- Huge reference databases (nr, Kraken, MEGAN, MetaPhlAn mpa_v20_m200, CheckM) dominate ~1 TB of disk and download/index time, and MEGAN's interactive installation requires user input, limiting full automation

## Evidence

- [methods] removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap: "removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap"
- [methods] Three assemblers (Megahit,SPAdes and IDBA) are integrated in this step: "Three assemblers (Megahit,SPAdes and IDBA) are integrated in this step"
- [methods] different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads: "different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads"
- [methods] After the stats, assembly filter is performed to filter contigs with minimum 1000bp length: "After the stats, assembly filter is performed to filter contigs with minimum 1000bp length"
- [readme] This review paper (https://doi.org/10.1093/bib/bbz155) aims to provide a comprehensive workflow to perform amplicon and shotgun metagenomics analysis.: "This review paper (https://doi.org/10.1093/bib/bbz155) aims to provide a comprehensive workflow to perform amplicon and shotgun metagenomics analysis."
- [abstract] we provide a set of best-practice workflows for 16S rRNA and metagenomic sequencing data (available at https://github.com/grimmlab/MicrobiomeBestPracticeReview): "we provide a set of best-practice workflows for 16S rRNA and metagenomic sequencing data (available at https://github.com/grimmlab/MicrobiomeBestPracticeReview)"
