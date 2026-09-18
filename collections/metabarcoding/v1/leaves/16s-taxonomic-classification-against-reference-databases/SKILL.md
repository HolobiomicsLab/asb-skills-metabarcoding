---
name: 16s-taxonomic-classification-against-reference-databases
description: Use when quality-filtered 16S rRNA amplicon reads (paired-end demultiplexed
  V4 fastq) or derived OTUs/ASVs need taxonomy assignment against curated 16S reference
  training sets within a mothur- or DADA2-based pipeline.
when_to_use_negative:
- Input is already a taxonomy table or feature/abundance table rather than raw or
  quality-filtered 16S reads — visualization/analysis, not classification, is needed
- Data are shotgun metagenomic reads rather than 16S rRNA amplicons — the review's
  metagenomics workflow (kraken2, MetaPhlAn2, DIAMOND, etc.) applies instead
- Reads are not paired-end, demultiplexed V4 amplicon data (e.g., long reads or non-demultiplexed
  runs), which the mothur/DADA2 scripts do not accept
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: mothur
  role: Clusters quality-filtered 16S reads into OTUs at the 97% identity threshold
    and classifies OTUs against Silva.nr_v132, Greengenes Gg_13_8_99, and RDP Trainset16_022016.pds
    reference sets
- name: DADA2
  role: R library implementing the denoising approach to infer error-free Amplicon
    Sequence Variants prior to taxonomy profiling
- name: Phyloseq
  role: R library used to summarize OTU/ASV community profiles and render taxonomic
    composition visualizations
- name: ggplot2
  role: R library used together with Phyloseq for taxonomic composition visualizations
- name: argparser
  role: R library used for argument handling in the visualization/summarization scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:903b057b22250aed0da47ffac5e59d38a905e7a12ef68064cb8d1354ccbd9223
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
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# 16s-taxonomic-classification-against-reference-databases

## Summary

Classify 16S rRNA amplicon reads, OTUs, or ASVs against curated 16S reference training sets (Silva nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds) as part of the best-practice amplicon workflow built on mothur (OTU path, 97% identity threshold) and DADA2 (ASV path).

## When to use

Use when quality-filtered 16S rRNA amplicon reads (paired-end demultiplexed V4 fastq) or derived OTUs/ASVs need taxonomy assignment against curated 16S reference training sets within a mothur- or DADA2-based pipeline.

## When NOT to use

- Input is already a taxonomy table or feature/abundance table rather than raw or quality-filtered 16S reads — visualization/analysis, not classification, is needed
- Data are shotgun metagenomic reads rather than 16S rRNA amplicons — the review's metagenomics workflow (kraken2, MetaPhlAn2, DIAMOND, etc.) applies instead
- Reads are not paired-end, demultiplexed V4 amplicon data (e.g., long reads or non-demultiplexed runs), which the mothur/DADA2 scripts do not accept

## Inputs

- Paired-end demultiplexed 16S rRNA V4 amplicon FASTQ files (e.g., Sample_1.fastq / Sample_2.fastq), quality-filtered
- Auto-downloaded 16S reference training sets: Silva.nr_v132, Greengenes Gg_13_8_99, RDP Trainset16_022016.pds
- Precompiled mothur binary and R with DADA2, Phyloseq, ggplot2, argparser libraries

## Outputs

- OTU clustering results at 97% identity threshold with taxonomic classifications (mothur path)
- Inferred Amplicon Sequence Variants (ASVs) with taxonomic assignments (DADA2 path)
- Non-empty OTU/ASV taxonomy tables and community-profile summaries
- Rendered taxonomic composition figure produced via Phyloseq/ggplot2

## How to apply

First quality-filter the raw paired-end demultiplexed V4 fastq files, then choose one of two alternative paths: an OTU-based path that clusters reads into operational taxonomic units at a predefined 97% identity threshold using mothur, or an ASV-based path that applies the DADA2 denoising approach to infer amplicon sequence variants before amplification/sequencing errors are introduced. Assign taxonomy to both OTU and ASV outputs against auto-downloaded 16S reference training sets: Silva.nr_v132, Greengenes Gg_13_8_99, and RDP Trainset16_022016.pds, using the precompiled mothur binary for the OTU path and the R DADA2 library for the ASV path. Summarize the resulting community profiles and generate taxonomic composition visualizations with the R libraries Phyloseq, ggplot2, and argparser, all orchestrated by 'amplicon_analysis.sh' which calls 'mothur_workflow.sh' and 'dada2_workflow.sh'. The rationale is that OTU clustering (97% threshold) and ASV denoising offer two complementary, best-practice computational paths for the same quality-filtered input, each anchored to well-established 16S reference databases.

## Related tools

- **mothur** (Clusters quality-filtered 16S reads into OTUs at the 97% identity threshold and classifies OTUs against Silva.nr_v132, Greengenes Gg_13_8_99, and RDP Trainset16_022016.pds reference sets)
- **DADA2** (R library implementing the denoising approach to infer error-free Amplicon Sequence Variants prior to taxonomy profiling)
- **Phyloseq** (R library used to summarize OTU/ASV community profiles and render taxonomic composition visualizations)
- **ggplot2** (R library used together with Phyloseq for taxonomic composition visualizations)
- **argparser** (R library used for argument handling in the visualization/summarization scripts)

## Examples

```
sh amplicon_analysis.sh  # with SRC_RAWDATA set to Sample_1.fastq/Sample_2.fastq and NAME set to the experiment folder
```

## Evaluation signals

- Both mothur_workflow.sh and dada2_workflow.sh complete on the mothur MiSeq SOP example data (2x250 V4 reads)
- Non-empty OTU and ASV taxonomy tables are produced for the example dataset
- A rendered phyloseq taxonomic composition figure is generated from the classification output
- OTU clustering uses the predefined 97% identity threshold as specified in the workflow
- Classification uses the auto-downloaded references Silva.nr_v132, Greengenes Gg_13_8_99, and RDP Trainset16_022016.pds

## Limitations

- Taxonomy assignment is reference-dependent: results differ between Silva nr_v132, Greengenes Gg_13_8_99, and RDP Trainset16 training sets
- ASV-based results depend on the DADA2 denoising model and assume errors are removed before variant inference; OTU results depend on the chosen 97% identity threshold
- The workflow is validated only on 2x250 V4 MiSeq SOP example reads; other read layouts or hypervariable regions are untested in this pipeline

## Evidence

- [methods] OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [methods] ASV based tool utilizes a denoising approach on biological sequences in samples: "ASV based tool utilizes a denoising approach on biological sequences in samples"
- [readme] First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data: "First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data."
- [readme] Both the workflows are controlled by bash scripts: amplicon_analysis.sh and metagenomics_analysis.sh: "Both the workflows are controlled by bash scripts: `amplicon_analysis.sh` and `metagenomics_analysis.sh`."
