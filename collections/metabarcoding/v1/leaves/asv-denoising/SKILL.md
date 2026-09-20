---
name: asv-denoising
description: Use when raw paired-end, demultiplexed 16S rRNA amplicon FASTQ data (e.g.
  V4 2x250 reads) must be converted to an error-corrected ASV table rather than clustered
  into OTUs at a 97% identity threshold.
when_to_use_negative:
- Input is already an ASV or OTU feature/taxonomy table — denoising operates on raw
  quality-filtered reads
- The data are shotgun metagenomic reads, which follow the 'metagenomics_analysis.sh'
  workflow (assembly, binning, kraken2/MetaPhlAn2 classification) rather than the
  amplicon workflow
- The analysis goal is OTU-based clustering at the predefined 97% identity threshold
  — use the mothur path ('mothur_workflow.sh') instead
edam_operation: http://edamontology.org/operation_3236
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: DADA2
  role: R library implementing the denoising approach that infers ASVs from quality-filtered
    amplicon reads
- name: Phyloseq
  role: R library used to summarize ASV community profiles and render taxonomic composition
    visualizations
- name: mothur
  role: Alternative OTU-based path in the same amplicon workflow; contrast option
    when 97% identity clustering is preferred over ASV denoising
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:2c57ec905ac31f7f84e6e3dddff8eaa5e3f027064984158711208b08681bad73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Phyloseq
  - argparser
  - ggplot2
  - mothur
  edam_operation: http://edamontology.org/operation_3236
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ASV denoising

## Summary

Infers Amplicon Sequence Variants (ASVs) from quality-filtered 16S rRNA amplicon reads using the denoising approach implemented in the R DADA2 library, which models sequencing errors so variants are corrected before amplification/sequencing errors propagate into downstream taxonomic profiles.

## When to use

Use when raw paired-end, demultiplexed 16S rRNA amplicon FASTQ data (e.g. V4 2x250 reads) must be converted to an error-corrected ASV table rather than clustered into OTUs at a 97% identity threshold.

## When NOT to use

- Input is already an ASV or OTU feature/taxonomy table — denoising operates on raw quality-filtered reads
- The data are shotgun metagenomic reads, which follow the 'metagenomics_analysis.sh' workflow (assembly, binning, kraken2/MetaPhlAn2 classification) rather than the amplicon workflow
- The analysis goal is OTU-based clustering at the predefined 97% identity threshold — use the mothur path ('mothur_workflow.sh') instead

## Inputs

- Paired-end, demultiplexed 16S rRNA amplicon FASTQ files (e.g. Sample_1.fastq / Sample_2.fastq, V4 region, 2x250 reads)
- Quality-filtered amplicon reads
- Curated 16S reference training sets (Silva.nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds)

## Outputs

- Amplicon Sequence Variant (ASV) table
- ASV taxonomy classification table
- Taxonomic composition visualizations rendered via Phyloseq/ggplot2

## How to apply

Configure 'amplicon_analysis.sh' with SRC_RAWDATA pointing at the paired-end demultiplexed V4 fastq files (Sample_1.fastq / Sample_2.fastq) and set NAME for the experiment folder; the script automatically calls 'dada2_workflow.sh'. Quality-filter raw reads first, then run the DADA2 denoising approach implemented in the R DADA2 library to infer Amplicon Sequence Variants before amplification/sequencing errors are introduced — this is the ASV-based alternative to the mothur OTU path that clusters reads at a predefined 97% identity threshold. Perform taxonomy profiling of the ASV output against the curated 16S reference training sets (Silva nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds) and summarize community profiles with Phyloseq, ggplot2 and argparser. Validate by confirming 'dada2_workflow.sh' completes on the mothur MiSeq SOP example data (2x250 V4 reads) and produces a non-empty ASV taxonomy table plus a rendered phyloseq figure.

## Related tools

- **DADA2** (R library implementing the denoising approach that infers ASVs from quality-filtered amplicon reads)
- **Phyloseq** (R library used to summarize ASV community profiles and render taxonomic composition visualizations)
- **mothur** (Alternative OTU-based path in the same amplicon workflow; contrast option when 97% identity clustering is preferred over ASV denoising)

## Examples

```
sh amplicon_analysis.sh
```

## Evaluation signals

- 'dada2_workflow.sh' completes end-to-end on the mothur MiSeq SOP example data (2x250 V4 reads) without errors
- A non-empty ASV taxonomy table is produced after classification against the curated 16S training sets
- A rendered phyloseq taxonomic composition figure is generated from the ASV output
- Both alternative paths (mothur OTU clustering and DADA2 ASV inference) produce comparable taxonomic profiles on the same input reads
- Input FASTQ files satisfy the workflow conventions: paired-end, demultiplexed, named *_1/_2

## Limitations

- The ASV path is one of two alternative branches; the workflow also requires an OTU-based path (mothur at 97% identity), and choosing between them is a decision point, not a default
- The workflow's bash scripts rely on a 'check_and_install' function and external reference databases (Silva nr_v132, Greengenes, RDP Trainset16) that must be downloaded before runs; missing tools or databases will make the workflow fail
- Only Linux (tested on Ubuntu 18.04.2) with R installed is supported for running the workflow scripts

## Evidence

- [methods] ASV based tool utilizes a denoising approach on biological sequences in samples: "ASV based tool utilizes a denoising approach on biological sequences in samples"
- [methods] OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [readme] First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided: "First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data."
- [readme] Both the workflows are controlled by bash scripts: `amplicon_analysis.sh` and `metagenomics_analysis.sh`: "Both the workflows are controlled by bash scripts: `amplicon_analysis.sh` and `metagenomics_analysis.sh`."
