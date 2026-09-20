---
name: best-practice-software-cataloging
description: Use when building or auditing a 16S rRNA amplicon or shotgun metagenomics
  pipeline and you need the review's recommended tool per phase (e.g. Trimmomatic/Sickle/BBMap
  for quality trimming, MEGAHIT/SPAdes/IDBA for assembly, MetaBAT/MaxBin for binning,
  Kraken2/MetaPhlAn2 for classification).
when_to_use_negative:
- The goal is to execute the pipeline on actual sequencing data rather than to catalogue
  which tools the review assigns to each phase — use the repository's amplicon_analysis.sh
  / metagenomics_analysis.sh scripts instead
- The analysis targets a non-microbiome data type (e.g. RNA-Seq or metabolomics) for
  which the review's 16S/metagenomic tool recommendations do not apply
- A tool inventory already exists and only a changelog or version history of the repository
  is needed — the review provides no changelog
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3697
tools:
- name: FastQC
  role: Generates a comprehensive quality report on raw FASTQ data as the QC entry
    point
  repo: https://github.com/s-andrews/FastQC/
- name: Trimmomatic
  role: Removal of adapters, low quality reads, sequencing artifacts, phix adapters
    and host contamination in the QC step
- name: Sickle
  role: Quality trimming alongside Trimmomatic and BBMap in the QC step
- name: BBMap
  role: Quality trimming and host contamination removal in the QC step
- name: MEGAHIT
  role: One of three assemblers integrated in the assembly step
- name: SPAdes
  role: One of three assemblers integrated in the assembly step
- name: IDBA
  role: One of three assemblers integrated in the assembly step
- name: MetaBAT
  role: Coverage-based binning of assembled contigs; installed from Bitbucket and
    expected in the system PATH
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- name: MaxBin
  role: Coverage-based binning of assembled contigs
- name: Binning_refiner
  role: Bin refinement after coverage-based binning
- name: CheckM
  role: Estimates genome completeness and contamination of bins via marker genes
  repo: https://github.com/Ecogenomics/CheckM/wiki/Installation
- name: kraken2
  role: State-of-the-art classification of output metagenomic reads
  repo: http://github.com/DerrickWood/kraken2/archive/v2.0.8-beta.tar.gz
- name: MetaPhlAn2
  role: Classification of output reads against the mpa_v20_m200 marker database (bowtie2-indexed)
  repo: https://bitbucket.org/biobakery/metaphlan2/downloads/mpa_v20_m200.tar
- name: DIAMOND
  role: Diamond blastx against the NR database for read classification/annotation
- name: MEGAN
  role: Meganized DIAMOND output used to classify reads; interactive install with
    3–16 GB memory option
  repo: http://ab.inf.uni-tuebingen.de/data/software/megan6/download/
- name: HUMAnN2
  role: Functional annotation/profiling of classified reads
- name: Prokka
  role: Annotation of assembled contigs (with Prodigal gene prediction)
- name: Prodigal
  role: Gene prediction step feeding Prokka annotation
- name: mothur
  role: OTU-based (97% identity threshold) 16S rRNA V4 amplicon workflow
- name: DADA2
  role: Denoising-based ASV workflow for 16S rRNA V4 amplicon data
- name: phyloseq
  role: Visualization of processed amplicon data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:cbf3a431fbbda8a3d8eb7991971eb806056a9dc208cde9f3a18a0c7936144e8b
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
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6956521739130435
n_claims: 23
---

# best-practice-software-cataloging

## Summary

Catalogues the concrete software tools that the Bharti & Grimm (2019, Briefings in Bioinformatics, doi:10.1093/bib/bbz155) best-practice workflows assign to each microbiome analysis phase (QC, assembly, binning, classification, annotation), producing a phase-to-tool map (best_practice_tools_map.csv) for reproducible pipeline planning.

## When to use

Use when building or auditing a 16S rRNA amplicon or shotgun metagenomics pipeline and you need the review's recommended tool per phase (e.g. Trimmomatic/Sickle/BBMap for quality trimming, MEGAHIT/SPAdes/IDBA for assembly, MetaBAT/MaxBin for binning, Kraken2/MetaPhlAn2 for classification).

## When NOT to use

- The goal is to execute the pipeline on actual sequencing data rather than to catalogue which tools the review assigns to each phase — use the repository's amplicon_analysis.sh / metagenomics_analysis.sh scripts instead
- The analysis targets a non-microbiome data type (e.g. RNA-Seq or metabolomics) for which the review's 16S/metagenomic tool recommendations do not apply
- A tool inventory already exists and only a changelog or version history of the repository is needed — the review provides no changelog

## Inputs

- Methods text of Bharti & Grimm 2019 (doi:10.1093/bib/bbz155) and the MicrobiomeBestPracticeReview repository README
- Named analysis phases of the metagenomic and amplicon workflows (quality control, assembly, binning, classification, annotation, visualization)

## Outputs

- best_practice_tools_map.csv with one row per analysis phase and columns: phase, data type (metagenomic/amplicon), best-practice tool(s), tool role

## How to apply

Parse the review's methods to enumerate each analysis phase of the metagenomic workflow (quality control, assembly, binning, taxonomic/read classification, functional annotation) and the two amplicon workflows (mothur and DADA2). For each phase, record only tools explicitly named in the text: FastQC plus Trimmomatic/Sickle/BBMap for adapter, low-quality-read, phix and host-contamination removal; MEGAHIT, SPAdes and IDBA as the three integrated assemblers; MetaBAT and MaxBin for coverage-based binning with Binning_refiner and CheckM for refinement; Kraken2, MetaPhlAn2, DIAMOND, MEGAN and HUMAnN2 for read classification and functional profiling; Prokka (with Prodigal gene prediction) for annotation. For amplicon data, distinguish OTU clustering at the common 97% identity threshold (mothur-style) from denoising-based ASV methods (DADA2). Emit one row per phase with columns phase, data type (metagenomic/amplicon), best-practice tool(s), and role, then validate that every methods step (steps 1–9 of the metagenomic workflow plus mothur/DADA2) is represented before writing best_practice_tools_map.csv.

## Related tools

- **FastQC** (Generates a comprehensive quality report on raw FASTQ data as the QC entry point) — https://github.com/s-andrews/FastQC/
- **Trimmomatic** (Removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination in the QC step)
- **Sickle** (Quality trimming alongside Trimmomatic and BBMap in the QC step)
- **BBMap** (Quality trimming and host contamination removal in the QC step)
- **MEGAHIT** (One of three assemblers integrated in the assembly step)
- **SPAdes** (One of three assemblers integrated in the assembly step)
- **IDBA** (One of three assemblers integrated in the assembly step)
- **MetaBAT** (Coverage-based binning of assembled contigs; installed from Bitbucket and expected in the system PATH) — https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- **MaxBin** (Coverage-based binning of assembled contigs)
- **Binning_refiner** (Bin refinement after coverage-based binning)
- **CheckM** (Estimates genome completeness and contamination of bins via marker genes) — https://github.com/Ecogenomics/CheckM/wiki/Installation
- **kraken2** (State-of-the-art classification of output metagenomic reads) — http://github.com/DerrickWood/kraken2/archive/v2.0.8-beta.tar.gz
- **MetaPhlAn2** (Classification of output reads against the mpa_v20_m200 marker database (bowtie2-indexed)) — https://bitbucket.org/biobakery/metaphlan2/downloads/mpa_v20_m200.tar
- **DIAMOND** (Diamond blastx against the NR database for read classification/annotation)
- **MEGAN** (Meganized DIAMOND output used to classify reads; interactive install with 3–16 GB memory option) — http://ab.inf.uni-tuebingen.de/data/software/megan6/download/
- **HUMAnN2** (Functional annotation/profiling of classified reads)
- **Prokka** (Annotation of assembled contigs (with Prodigal gene prediction))
- **Prodigal** (Gene prediction step feeding Prokka annotation)
- **mothur** (OTU-based (97% identity threshold) 16S rRNA V4 amplicon workflow)
- **DADA2** (Denoising-based ASV workflow for 16S rRNA V4 amplicon data)
- **phyloseq** (Visualization of processed amplicon data)

## Examples

```
sh metagenomics_analysis.sh  # after setting LINKPATH_DB, SRC_RAWDATA and NAME; see also run_qc.sh for the quality-control step
```

## Evaluation signals

- best_practice_tools_map.csv contains a row for each phase of the metagenomic workflow (steps 1–9) plus separate mothur and DADA2 amplicon rows
- QC row lists FastQC with Trimmomatic/Sickle/BBMap, matching the paper's statement that adapter/low-quality/host-contamination removal 'is performed using trimmomatic, sickle and bbmap'
- Assembly row lists exactly the three integrated assemblers MEGAHIT, SPAdes and IDBA
- Binning rows include both coverage-based binners (MetaBAT, MaxBin) and refinement tools (Binning_refiner, CheckM)
- Each row's data type column is correctly set to metagenomic or amplicon, and every named tool has an assigned role

## Limitations

- The review's workflows were tested only on Ubuntu 18.04.2 (bionic) and demand heavy hardware (16+ cores, ~100 GB RAM, ~1 TB disk, mostly for nr/kraken reference databases)
- MEGAN installation is interactive (Y/N prompts and a memory choice of ~3–16 GB depending on sample size), so the catalogue alone does not guarantee unattended installation
- Paired-end, demultiplexed, gzipped Illumina reads ending in *_1.fastq.gz/*_2.fastq.gz are assumed; other input layouts are not covered
- CheckM (v1) is flagged as unmaintained and relies on a manually installed checkM database; the repository provides no changelog

## Evidence

- [methods] FastQC quality report on raw data: "[FastQC](https://github.com/s-andrews/FastQC/) is used to generate comprehensive report of data quality on raw data"
- [methods] QC trimming and decontamination tools: "removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap"
- [methods] Three integrated assemblers: "Three assemblers (Megahit,SPAdes and IDBA) are integrated in this step"
- [methods] Read classification tools: "different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads"
- [methods] OTU clustering threshold: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [readme] Workflows provided as runnable scripts: "Both the workflows are controlled by bash scripts: `amplicon_analysis.sh` and `metagenomics_analysis.sh`."
