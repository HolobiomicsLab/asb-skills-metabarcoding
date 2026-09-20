---
name: 16s-amplicon-otu-clustering
description: Use when raw demultiplexed paired-end 16S rRNA amplicon FASTQ reads (e.g.
  V4 2x250) must be clustered into operational taxonomic units at the 97% identity
  threshold with mothur, with taxonomy assigned against Silva/Greengenes/RDP training
  sets.
when_to_use_negative:
- Input is already an OTU/ASV feature table or taxonomy table rather than raw demultiplexed
  paired-end amplicon FASTQ reads
- The analysis target is shotgun metagenomics data — use the review's metagenomics
  workflow (metagenomics_analysis.sh) instead of the amplicon workflow
- Reads are not paired-end or not demultiplexed, since the workflow requires paired-end
  demultiplexed Illumina reads
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: mothur
  role: Clusters quality-filtered 16S reads into OTUs at the predefined 97% identity
    threshold and classifies them against Silva/Greengenes/RDP training sets
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
- name: DADA2
  role: Alternative ASV-based path in the amplicon workflow, denoising biological
    sequences to infer Amplicon Sequence Variants
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
- name: Phyloseq
  role: R library used to summarize OTU/ASV community profiles and build taxonomic
    composition visualizations
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
- name: ggplot2
  role: R library used with Phyloseq for taxonomic composition plotting
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
- name: argparser
  role: R library used for argument parsing in the visualization scripts
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:d248656a4d64f81af2370a82061444ccb553fbd2f4f9e67a411ce12aa9b61427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/grimmlab/MicrobiomeBestPracticeReview
  tools:
  - DADA2
  - Phyloseq
  - argparser
  - ggplot2
  - mothur
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# 16s-amplicon-otu-clustering

## Summary

Best-practice 16S rRNA amplicon workflow that quality-filters raw paired-end reads and clusters them into OTUs at a predefined 97% identity threshold using mothur, as the OTU-based alternative to DADA2's ASV denoising path, followed by taxonomy profiling and phyloseq/ggplot2 visualization.

## When to use

Use when raw demultiplexed paired-end 16S rRNA amplicon FASTQ reads (e.g. V4 2x250) must be clustered into operational taxonomic units at the 97% identity threshold with mothur, with taxonomy assigned against Silva/Greengenes/RDP training sets.

## When NOT to use

- Input is already an OTU/ASV feature table or taxonomy table rather than raw demultiplexed paired-end amplicon FASTQ reads
- The analysis target is shotgun metagenomics data — use the review's metagenomics workflow (metagenomics_analysis.sh) instead of the amplicon workflow
- Reads are not paired-end or not demultiplexed, since the workflow requires paired-end demultiplexed Illumina reads

## Inputs

- Paired-end demultiplexed 16S rRNA V4 FASTQ files (Sample_1.fastq / Sample_2.fastq; *.fastq.gz pairs ending in *_1.fastq.gz and *_2.fastq.gz)
- mothur MiSeq SOP example data (2x250 V4 reads) for validation
- Reference training sets: Silva.nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds

## Outputs

- Non-empty OTU clustering and taxonomy tables at the 97% identity threshold
- ASV tables from the parallel DADA2 denoising path
- Rendered taxonomic composition figure from Phyloseq/ggplot2

## How to apply

Configure and run 'amplicon_analysis.sh' with SRC_RAWDATA pointing at paired-end demultiplexed V4 fastq files named Sample_1.fastq/Sample_2.fastq and NAME set for the experiment folder; the script automatically invokes 'mothur_workflow.sh' and 'dada2_workflow.sh' and each sub-script runs its own 'check_and_install' to fetch missing tools. In the mothur OTU path, quality-filter the reads first, then cluster them at the predefined 97% identity threshold into operational taxonomic units using the precompiled mothur binary. Classify the resulting OTUs against the auto-downloaded references Silva.nr_v132 (mothur.org/zenodo 1172783), Greengenes Gg_13_8_99, and RDP Trainset16_022016.pds. Run the DADA2 path in parallel, which applies denoising on biological sequences to infer ASVs before amplification/sequencing errors are introduced. Summarize the OTU and ASV community profiles and render taxonomic composition figures with the R libraries Phyloseq, ggplot2, and argparser. The rationale is the review's aim to standardize 16S rRNA analysis for robustness and reproducibility.

## Related tools

- **mothur** (Clusters quality-filtered 16S reads into OTUs at the predefined 97% identity threshold and classifies them against Silva/Greengenes/RDP training sets) — https://github.com/grimmlab/MicrobiomeBestPracticeReview
- **DADA2** (Alternative ASV-based path in the amplicon workflow, denoising biological sequences to infer Amplicon Sequence Variants) — https://github.com/grimmlab/MicrobiomeBestPracticeReview
- **Phyloseq** (R library used to summarize OTU/ASV community profiles and build taxonomic composition visualizations) — https://github.com/grimmlab/MicrobiomeBestPracticeReview
- **ggplot2** (R library used with Phyloseq for taxonomic composition plotting) — https://github.com/grimmlab/MicrobiomeBestPracticeReview
- **argparser** (R library used for argument parsing in the visualization scripts) — https://github.com/grimmlab/MicrobiomeBestPracticeReview

## Examples

```
sh amplicon_analysis.sh
```

## Evaluation signals

- Both mothur_workflow.sh and dada2_workflow.sh complete without error on the mothur MiSeq SOP example data (2x250 V4 reads)
- Non-empty OTU and ASV taxonomy tables are produced after classification against Silva.nr_v132 / Gg_13_8_99 / Trainset16_022016.pds
- A rendered phyloseq taxonomic composition figure is generated as final output
- OTU clustering uses the predefined 97% identity threshold rather than an ad hoc cutoff
- The 'check_and_install' function confirms all required tools exist in the expected system path before the pipeline runs

## Limitations

- Requires Linux (tested on Ubuntu 18.04.2) and that all tools exist in the system beforehand; installation of the many dependencies is time-consuming and the workflow can fail if external software is missing
- Reference databases (Silva.nr_v132, Greengenes, RDP Trainset16) must be downloaded and can be large, slowing setup
- Databases are downloaded automatically and indexed on the fly, so users with pre-existing databases must edit the scripts with correct paths
- The two paths are alternative computational routes (OTU-based mothur vs ASV-based DADA2); results differ by construction and are not interchangeable

## Evidence

- [methods] OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [methods] ASV denoising alternative: "ASV based tool utilizes a denoising approach on biological sequences in samples"
- [readme] Two amplicon workflows mothur and dada2: "First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data."
- [readme] check_and_install pattern: "The bash scripts contain functions which call the respective underlying tools."
- [methods] Reference training sets for classification: "https://mothur.org/w/images/3/32/Silva.nr_v132.tgz
https://zenodo.org/record/1172783/files/silva_nr_v132_train_set.fa.gz"
