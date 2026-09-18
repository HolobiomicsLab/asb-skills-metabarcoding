---
name: paired-end-amplicon-read-workflow-scripting
description: Use when raw paired-end, demultiplexed 16S rRNA amplicon FASTQ files
  (e.g. V4 region Sample_1.fastq/Sample_2.fastq) must be turned into OTU or ASV community
  profiles with taxonomic assignments and plots.
when_to_use_negative:
- Input is shotgun metagenomic data — use the review's metagenomics workflow (metagenomics_analysis.sh)
  with assemblers and binning instead of OTU/ASV clustering.
- Input is already an OTU/ASV abundance table — this skill processes raw reads, not
  feature tables.
- Reads are single-end or not demultiplexed — the workflow requires paired-end, demultiplexed
  reads.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: mothur
  role: 'OTU-based path: clusters quality-filtered reads into OTUs at the 97% identity
    threshold and classifies them against Silva, Greengenes, and RDP references'
- name: DADA2
  role: 'ASV-based path: R library implementing denoising to infer Amplicon Sequence
    Variants before amplification/sequencing errors are introduced'
- name: Phyloseq
  role: Summarizes community profiles and renders taxonomic composition visualizations
- name: ggplot2
  role: Graphics library used by Phyloseq for taxonomic composition plots
- name: argparser
  role: R argument parsing used by the workflow's visualization scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:2da6a19088c3126df70995ee445075095c336fa439a0d07b41e146b7d1fba6ae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Phyloseq
  - argparser
  - ggplot2
  - mothur
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# paired-end-amplicon-read-workflow-scripting

## Summary

A best-practice 16S rRNA amplicon analysis skill that quality-filters raw paired-end reads and branches them into an OTU-based path (mothur, 97% identity threshold) or an ASV-based path (DADA2 denoising), followed by taxonomy profiling and phyloseq/ggplot2 visualization.

## When to use

Use when raw paired-end, demultiplexed 16S rRNA amplicon FASTQ files (e.g. V4 region Sample_1.fastq/Sample_2.fastq) must be turned into OTU or ASV community profiles with taxonomic assignments and plots.

## When NOT to use

- Input is shotgun metagenomic data — use the review's metagenomics workflow (metagenomics_analysis.sh) with assemblers and binning instead of OTU/ASV clustering.
- Input is already an OTU/ASV abundance table — this skill processes raw reads, not feature tables.
- Reads are single-end or not demultiplexed — the workflow requires paired-end, demultiplexed reads.

## Inputs

- Paired-end demultiplexed 16S rRNA amplicon FASTQ files (e.g. Sample_1.fastq / Sample_2.fastq, V4 region)
- Silva reference database (nr_v132, auto-downloaded from mothur.org / Zenodo)
- Greengenes database (Gg_13_8_99 taxonomy)
- RDP database (Trainset16_022016.pds)

## Outputs

- OTU cluster tables (97% identity threshold) with taxonomy assignments from mothur
- ASV tables with taxonomy assignments from DADA2
- Taxonomic composition visualizations rendered with Phyloseq and ggplot2

## How to apply

Configure and run 'amplicon_analysis.sh' with SRC_RAWDATA pointing to the paired-end demultiplexed V4 fastq files and NAME set for the experiment folder; the script automatically calls 'mothur_workflow.sh' and 'dada2_workflow.sh'. In the mothur OTU path, cluster quality-filtered reads into operational taxonomic units using the predefined 97% identity threshold and classify against auto-downloaded references Silva.nr_v132, Greengenes Gg_13_8_99, and RDP Trainset16_022016.pds using the precompiled mothur binary. In the DADA2 path, run the denoising approach in the R DADA2 library to infer Amplicon Sequence Variants before amplification/sequencing errors are introduced. Then profile taxonomy of both OTU and ASV outputs against the curated 16S reference training sets and summarize community profiles with Phyloseq, ggplot2, and argparser.

## Related tools

- **mothur** (OTU-based path: clusters quality-filtered reads into OTUs at the 97% identity threshold and classifies them against Silva, Greengenes, and RDP references)
- **DADA2** (ASV-based path: R library implementing denoising to infer Amplicon Sequence Variants before amplification/sequencing errors are introduced)
- **Phyloseq** (Summarizes community profiles and renders taxonomic composition visualizations)
- **ggplot2** (Graphics library used by Phyloseq for taxonomic composition plots)
- **argparser** (R argument parsing used by the workflow's visualization scripts)

## Examples

```
SRC_RAWDATA=/path/Sample_1.fastq NAME=amplicon_exp; sh amplicon_analysis.sh
```

## Evaluation signals

- Both mothur_workflow.sh and dada2_workflow.sh complete successfully on the mothur MiSeq SOP example data (2x250 V4 reads)
- Non-empty OTU and ASV taxonomy tables are produced after classification against the reference training sets
- A rendered phyloseq taxonomic composition figure is generated from the processed data
- OTU clustering is verified against the predefined 97% identity threshold parameter
- Raw data are validated as paired-end, demultiplexed, *.fastq.gz with *_1.fastq.gz / *_2.fastq.gz naming before the workflow starts

## Limitations

- The tools must already exist in the system or be installed on the fly via the check_and_install function; download and installation of the many tools and large reference databases takes considerable time.
- The workflow was tested only on Linux (Ubuntu 18.04.2 LTS); other operating systems are not validated.
- Sample reads must always be paired-end, demultiplexed, and compressed in *.fastq.gz format — otherwise the workflow will fail.

## Evidence

- [methods] OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [methods] ASV denoising: "ASV based tool utilizes a denoising approach on biological sequences in samples"
- [readme] README workflow description: "First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data."
- [readme] README input format requirement: "The sample reads must always be paired-end, demultiplexed and compressed in the\*.fastq.gz format."
- [abstract] Abstract workflow scope: "we provide a set of best-practice workflows for 16S rRNA and metagenomic sequencing data (available at https://github.com/grimmlab/MicrobiomeBestPracticeReview)"
