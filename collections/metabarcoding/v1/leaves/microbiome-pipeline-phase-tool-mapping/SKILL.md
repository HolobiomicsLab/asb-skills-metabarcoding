---
name: microbiome-pipeline-phase-tool-mapping
description: Use when building or auditing a 16S rRNA or shotgun metagenomics pipeline
  and you need to know which concrete tool (e.g. Trimmomatic, MEGAHIT, MetaBAT, Kraken2,
  Prokka, mothur, DADA2) the highest-ranked-practice literature assigns to each phase.
when_to_use_negative:
- You already have a validated pipeline and only need runtime execution instructions
  rather than tool selection per phase
- The task concerns long-read metagenomics tool choices not explicitly covered by
  the short-read tool catalogue surfaced here
- Input is a processed feature table or abundance matrix where phase-level tool selection
  is no longer relevant
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: FastQC
  role: Generates a comprehensive report of data quality on raw metagenomic reads
    before trimming
  repo: https://github.com/s-andrews/FastQC/
- name: Trimmomatic
  role: 'Quality trimming: removal of adapters, low quality reads, sequencing artifacts,
    phix adapters and host contamination'
- name: Sickle
  role: Quality trimming alongside Trimmomatic and BBMap in the QC step (run_qc.sh)
- name: BBMap
  role: Quality trimming and host contamination removal in the QC step
- name: MEGAHIT
  role: One of three integrated metagenomic assemblers for the assembly phase
- name: SPAdes
  role: One of three integrated metagenomic assemblers for the assembly phase
- name: IDBA
  role: One of three integrated metagenomic assemblers for the assembly phase
- name: MetaBAT
  role: Coverage-based binning of assembled contigs
  repo: https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- name: CheckM
  role: Bin refinement/quality assessment of metagenome-assembled genomes
  repo: https://github.com/Ecogenomics/CheckM/wiki/Installation
- name: kraken2
  role: State-of-the-art classification of output reads
  repo: http://github.com/DerrickWood/kraken2/archive/v2.0.8-beta.tar.gz
- name: MetaPhlAn2
  role: State-of-the-art classification of output reads (database mpa_v20_m200 built
    with bowtie2)
  repo: https://bitbucket.org/biobakery/metaphlan2/downloads/mpa_v20_m200.tar
- name: DIAMOND
  role: DIAMOND blastx against the NR database for read classification
- name: MEGAN
  role: Meganization of DIAMOND blastx output for classification
  repo: http://ab.inf.uni-tuebingen.de/data/software/megan6/download/
- name: HUMAnN2
  role: Functional profiling/classification of output reads
- name: Prokka
  role: Annotation of assembled contigs (with Prodigal gene prediction)
- name: mothur
  role: OTU-based amplicon workflow with 97% identity threshold clustering using Silva/Greengenes/RDP
    references
- name: DADA2
  role: ASV denoising-based amplicon workflow as the alternative to OTU clustering
- name: phyloseq
  role: Visualization of processed amplicon data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:f47f48251c580af263b249fb7b85466bbd96dfe45589cc5069e5af88acf8bdb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/s-andrews/FastQC
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
  - phyloseq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 25
---

# microbiome-pipeline-phase-tool-mapping

## Summary

Constructs a phase-by-phase catalogue of the best-practice tools recommended by the Bharti & Grimm (2019) microbiome review, mapping each metagenomic and amplicon analysis phase (quality control, assembly, binning, read classification, annotation, visualization) to the specific named software and its role.

## When to use

Use when building or auditing a 16S rRNA or shotgun metagenomics pipeline and you need to know which concrete tool (e.g. Trimmomatic, MEGAHIT, MetaBAT, Kraken2, Prokka, mothur, DADA2) the best-practice literature assigns to each phase.

## When NOT to use

- You already have a validated pipeline and only need runtime execution instructions rather than tool selection per phase
- The task concerns long-read metagenomics tool choices not explicitly covered by the short-read tool catalogue surfaced here
- Input is a processed feature table or abundance matrix where phase-level tool selection is no longer relevant

## Inputs

- Raw paired-end, demultiplexed Illumina metagenomic reads in *.fastq.gz (e.g. _1.fastq.gz/_2.fastq.gz pairs from the metaHIT gut survey, ENA ERR011)
- Raw 16S rRNA amplicon sequencing reads (V4 region) for the mothur and DADA2 workflows
- Reference databases: Silva nr_v132, Greengenes gg_13_8_99, RDP Trainset16_022016.pds, Kraken database, Megan database, NR database, MetaPhlAn mpa_v20_m200, checkM database, Humann2 database
- The review's methods text (Bharti & Grimm, doi:10.1093/bib/bbz155) and the MicrobiomeBestPracticeReview repository scripts

## Outputs

- A phase-to-tool mapping table (best_practice_tools_map.csv) with columns: phase, data type (metagenomic/amplicon), best-practice tool(s), and the tool's role
- Phase-level annotations of which named tool covers quality control, assembly, binning, bin refinement, taxonomic classification, functional annotation, and visualization

## How to apply

Parse the review's methods workflow (steps 1–9 of the metagenomic pipeline plus the mothur and DADA2 amplicon workflows) and record one row per phase. For metagenomic QC, FastQC generates the data-quality report on raw reads, then adapter, low-quality-read, sequencing-artifact, phix-adapter and host contamination removal is done with Trimmomatic, Sickle and BBMap; assembly uses the three integrated assemblers MEGAHIT, SPAdes and IDBA, followed by a filter keeping contigs of minimum 1000bp. Coverage-based binning uses MetaBAT and MaxBin with refinement via Binning_refiner and CheckM; read classification uses Kraken2, MetaPhlAn2, DIAMOND blastx against the NR database (meganized with MEGAN databases) and HUMAnN2; annotation uses Prokka with Prodigal gene prediction. For amplicon data, distinguish OTU clustering at the commonly used 97% identity threshold (mothur-style, with Silva nr_v132, Greengenes gg_13_8_99 or RDP Trainset16 references) from ASV denoising (DADA2), and note phyloseq for downstream visualization. Validate that every phase listed in the methods is represented in the final table (e.g. best_practice_tools_map.csv) with columns for phase, data type, tool(s), and role.

## Related tools

- **FastQC** (Generates a comprehensive report of data quality on raw metagenomic reads before trimming) — https://github.com/s-andrews/FastQC/
- **Trimmomatic** (Quality trimming: removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination)
- **Sickle** (Quality trimming alongside Trimmomatic and BBMap in the QC step (run_qc.sh))
- **BBMap** (Quality trimming and host contamination removal in the QC step)
- **MEGAHIT** (One of three integrated metagenomic assemblers for the assembly phase)
- **SPAdes** (One of three integrated metagenomic assemblers for the assembly phase)
- **IDBA** (One of three integrated metagenomic assemblers for the assembly phase)
- **MetaBAT** (Coverage-based binning of assembled contigs) — https://bitbucket.org/berkeleylab/metabat/src/master/README.md
- **CheckM** (Bin refinement/quality assessment of metagenome-assembled genomes) — https://github.com/Ecogenomics/CheckM/wiki/Installation
- **kraken2** (State-of-the-art classification of output reads) — http://github.com/DerrickWood/kraken2/archive/v2.0.8-beta.tar.gz
- **MetaPhlAn2** (State-of-the-art classification of output reads (database mpa_v20_m200 built with bowtie2)) — https://bitbucket.org/biobakery/metaphlan2/downloads/mpa_v20_m200.tar
- **DIAMOND** (DIAMOND blastx against the NR database for read classification)
- **MEGAN** (Meganization of DIAMOND blastx output for classification) — http://ab.inf.uni-tuebingen.de/data/software/megan6/download/
- **HUMAnN2** (Functional profiling/classification of output reads)
- **Prokka** (Annotation of assembled contigs (with Prodigal gene prediction))
- **mothur** (OTU-based amplicon workflow with 97% identity threshold clustering using Silva/Greengenes/RDP references)
- **DADA2** (ASV denoising-based amplicon workflow as the alternative to OTU clustering)
- **phyloseq** (Visualization of processed amplicon data)

## Examples

```
sh metagenomics_analysis.sh  # after prepare_databases.sh and setting LINKPATH_DB, SRC_RAWDATA and NAME in the script
```

## Evaluation signals

- Every phase named in the review's methods (steps 1–9 of the metagenomic workflow plus the mothur and DADA2 amplicon workflows) appears as a row in the mapping table
- The QC row lists all three trimming tools (Trimmomatic, Sickle, BBMap) and FastQC for the initial data-quality report, matching the 'trimmomatic, sickle and bbmap' evidence
- The assembly row lists all three integrated assemblers (Megahit, SPAdes, IDBA) and records the 1000bp minimum contig filter
- The amplicon section distinguishes OTU clustering (97% identity threshold) from ASV denoising and names both mothur and DADA2
- Each row has a non-empty data-type column (metagenomic vs amplicon) and a role description consistent with the cited evidence text

## Limitations

- Tool-to-phase assignments reflect the review's 2019 best-practice recommendations; e.g. CheckM v1 is listed although CheckM v2 now exists as a more accurate ML-based alternative
- The databases required (NR, Kraken, Megan, checkM, etc.) are huge and take a long time to download and index, and the workflows assume Linux, 16+ cores, ~100 GB RAM and 1 TB diskspace
- The review notes no concrete named software tools or versions are specified in the discussion itself; tool assignments come from the methods workflow, and 'optimally tested tools' are referenced only generically there

## Evidence

- [methods] FastQC generates comprehensive quality report: "[FastQC](https://github.com/s-andrews/FastQC/) is used to generate comprehensive report of data quality on raw data"
- [methods] Trimming and contamination removal tools: "removal of adapters, low quality reads, sequencing artifacts, phix adapters and host contamination is performed using trimmomatic, sickle and bbmap"
- [methods] Three integrated assemblers: "Three assemblers (Megahit,SPAdes and IDBA) are integrated in this step"
- [methods] Read classification tools: "different state of art tools like kraken2 and metaphlan2, Diamond, Megan and Humann2 are used to classify the output reads"
- [methods] OTU vs ASV distinction: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [methods] Assembly contig length filter: "After the stats, assembly filter is performed to filter contigs with minimum 1000bp length"
- [readme] Workflows provided in the repository: "we provide a set of best-practice workflows for 16S rRNA and metagenomic sequencing data (available at https://github.com/grimmlab/MicrobiomeBestPracticeReview)"
- [readme] Metagenomic QC is step 1 of the workflow: [evidence span withheld — anchor status: not_found]
