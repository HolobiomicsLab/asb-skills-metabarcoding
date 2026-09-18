---
name: microbiome-community-visualization-phyloseq
description: Use when quality-filtered 16S rRNA amplicon reads have been processed
  into OTU (97% identity, mothur) or ASV (DADA2 denoising) tables with taxonomy assignments,
  and taxonomic composition visualizations are required.
when_to_use_negative:
- Input is shotgun metagenomic data, which belongs to the separate metagenomics_analysis.sh
  workflow rather than the amplicon visualization path
- Input is raw unprocessed reads with no OTU/ASV taxonomy tables yet — run mothur_workflow.sh
  and dada2_workflow.sh first
- Data is already a rendered figure or pre-summarized community profile requiring
  no further Phyloseq processing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Phyloseq
  role: R library used to build phyloseq objects from OTU/ASV taxonomy tables and
    generate the taxonomic composition figure
- name: ggplot2
  role: R plotting library used for rendering taxonomic composition visualizations
- name: argparser
  role: R library providing command-line argument handling for the visualization scripts
- name: mothur
  role: Upstream OTU-based path producing the 97% identity clustered OTU tables that
    feed visualization
- name: DADA2
  role: Upstream ASV-based path producing denoised ASV tables that feed visualization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:e0a37b2dba033624c52837958f93269aa574505cf9b58ac78e0a8bbdae52b212
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Phyloseq
  - argparser
  - ggplot2
  - mothur
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# microbiome-community-visualization-phyloseq

## Summary

A skill for summarizing and visualizing 16S rRNA amplicon community profiles by rendering taxonomic composition figures from OTU (mothur) and ASV (DADA2) outputs using the R libraries Phyloseq, ggplot2, and argparser, as part of the best-practice amplicon workflow.

## When to use

Use when quality-filtered 16S rRNA amplicon reads have been processed into OTU (97% identity, mothur) or ASV (DADA2 denoising) tables with taxonomy assignments, and taxonomic composition visualizations are required.

## When NOT to use

- Input is shotgun metagenomic data, which belongs to the separate metagenomics_analysis.sh workflow rather than the amplicon visualization path
- Input is raw unprocessed reads with no OTU/ASV taxonomy tables yet — run mothur_workflow.sh and dada2_workflow.sh first
- Data is already a rendered figure or pre-summarized community profile requiring no further Phyloseq processing

## Inputs

- Non-empty OTU taxonomy tables from the mothur 97% identity path
- Non-empty ASV taxonomy tables from the DADA2 denoising path
- 16S reference training sets used for classification (Silva.nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds)
- Paired-end demultiplexed V4 fastq files (e.g. Sample_1.fastq / Sample_2.fastq) upstream input to the amplicon workflow

## Outputs

- Rendered taxonomic composition figure produced via Phyloseq/ggplot2
- Summarized community profiles for both OTU and ASV outputs

## How to apply

First run the amplicon workflow so that quality-filtered paired-end V4 fastq reads (Sample_1.fastq / Sample_2.fastq) are processed along both paths: mothur OTU clustering at the predefined 97% identity threshold and DADA2 ASV inference, each classified against the curated 16S reference training sets (Silva.nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds). Then, in R, load the resulting OTU and ASV taxonomy tables into Phyloseq to build phyloseq objects, summarize the community profiles, and render taxonomic composition plots with ggplot2, wrapping the R code with argparser for command-line use. This visualization step is the final stage of the amplicon workflow (step 5 of amplicon_analysis.sh, which calls mothur_workflow.sh and dada2_workflow.sh). Correct application is judged by confirming both workflow scripts complete on the mothur MiSeq SOP example data (2x250 V4 reads) and that non-empty OTU/ASV taxonomy tables plus a rendered phyloseq figure are produced.

## Related tools

- **Phyloseq** (R library used to build phyloseq objects from OTU/ASV taxonomy tables and generate the taxonomic composition figure)
- **ggplot2** (R plotting library used for rendering taxonomic composition visualizations)
- **argparser** (R library providing command-line argument handling for the visualization scripts)
- **mothur** (Upstream OTU-based path producing the 97% identity clustered OTU tables that feed visualization)
- **DADA2** (Upstream ASV-based path producing denoised ASV tables that feed visualization)

## Examples

```
SRC_RAWDATA=/path/to/fastq NAME=MY_AMPLICON_EXP; sh amplicon_analysis.sh
```

## Evaluation signals

- mothur_workflow.sh and dada2_workflow.sh both complete on the mothur MiSeq SOP example data (2x250 V4 reads)
- Non-empty OTU and ASV taxonomy tables are produced before visualization
- A rendered phyloseq figure is produced as the final output
- Both the mothur (97% identity OTU) and DADA2 (ASV) paths are represented in the summarized community profiles
- Visualizations derive from reads classified against the curated 16S reference training sets (Silva.nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds)

## Limitations

- Requires the underlying tools to exist in the system beforehand; the check_and_install function installs missing tools but download and installation can take considerable time
- Tested on Ubuntu 18.04.2 (bionic) with Python 3.7; other environments are untested
- Visualization depends on correct upstream taxonomy assignment; errors in quality filtering or classification propagate into the figures

## Evidence

- [full_text] Summarize community profiles and generate taxonomic composition visualizations using R libraries Phyloseq, ggplot2, and argparser.: [evidence span withheld — anchor status: not_found]
- [methods] OTU clustering at 97% identity threshold: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [readme] README: workflow description: "First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data."
- [readme] README: workflow control scripts: "Both the workflows are controlled by bash scripts: `amplicon_analysis.sh` and `metagenomics_analysis.sh`."
- [methods] ASV denoising input: "ASV based tool utilizes a denoising approach on biological sequences in samples"
