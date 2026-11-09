---
name: amplicon-otu-asv-method-differentiation
description: Use when raw 16S rRNA amplicon sequencing reads must be converted into
  OTUs or ASVs and the practitioner must decide between (or document both) the 97%-identity
  clustering route and the denoising ASV route.
when_to_use_negative:
- Input is shotgun metagenomic read data — use the review's metagenomic read-classification
  tools (Kraken2, MetaPhlAn2, DIAMOND/MEGAN, HUMAnN2) rather than OTU/ASV amplicon
  methods.
- The OTU/ASV abundance table already exists and only downstream comparative/statistical
  analysis or visualization is needed.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: mothur
  role: 'OTU-based amplicon workflow: clusters 16S rRNA reads into operational taxonomic
    units at the 97% identity threshold using reference taxonomy databases (Silva
    nr_v132, Greengenes, RDP)'
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
- name: DADA2
  role: 'ASV-based amplicon workflow: applies a denoising approach on biological sequences
    to infer exact amplicon sequence variants instead of clustering'
  repo: https://github.com/grimmlab/MicrobiomeBestPracticeReview
- name: phyloseq
  role: Standard visualization of processed OTU/ASV amplicon data in the best-practice
    workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbz155
    title: Current challenges and best-practice protocols for microbiome analysis
  source_content_hash: sha256:6b7a1b26b3dcb732ac396884f58744fc438d985d4aa14b8aa00594c16bdb35fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/grimmlab/MicrobiomeBestPracticeReview
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 25
---

# amplicon-otu-asv-method-differentiation

## Summary

Distinguishes the two 16S rRNA amplicon analysis paradigms described in Bharti & Grimm (doi:10.1093/bib/bbz155): OTU clustering of reads at a predefined identity threshold (commonly 97%, mothur-style) versus denoising-based ASV inference (DADA2), each with its own best-practice workflow and reference taxonomy databases.

## When to use

Use when raw 16S rRNA amplicon sequencing reads must be converted into OTUs or ASVs and the practitioner must decide between (or document both) the 97%-identity clustering route and the denoising ASV route.

## When NOT to use

- Input is shotgun metagenomic read data — use the review's metagenomic read-classification tools (Kraken2, MetaPhlAn2, DIAMOND/MEGAN, HUMAnN2) rather than OTU/ASV amplicon methods.
- The OTU/ASV abundance table already exists and only downstream comparative/statistical analysis or visualization is needed.

## Inputs

- Demultiplexed raw 16S rRNA amplicon (e.g., V4 region) sequencing reads in FASTQ format
- mothur-compatible reference taxonomy databases: Silva nr_v132 (silva.nr_v132.tgz), Greengenes Gg_13_8_99.taxonomy.tgz, or RDP Trainset16_022016.pds.tgz

## Outputs

- OTU count table (mothur workflow, 97% identity clustering)
- ASV sequence/abundance table (DADA2 denoising workflow)
- Taxonomy assignments for OTUs/ASVs against Silva, Greengenes, or RDP references
- Standard visualization outputs for the processed amplicon data (e.g., phyloseq-based plots)
- best_practice_tools_map.csv row recording phase = OTU clustering / ASV denoising, data type = amplicon, tool, and role

## How to apply

Decide first whether the analysis needs OTU-based or ASV-based outputs: OTU-based methods cluster reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units, while ASV-based tools utilize a denoising approach on biological sequences in samples. For the mothur route, classify V4 16S rRNA reads against a curated reference taxonomy such as Silva (silva.nr_v132), Greengenes (Gg_13_8_99), or the RDP training set (Trainset16_022016.pds), then cluster at 97% identity. For the DADA2 route, run the denoising workflow to infer exact amplicon sequence variants instead of clustering. The review supplies both mothur and DADA2 workflows for V4 16S rRNA data plus standard visualizations of the processed tables, all driven by bash scripts in the MicrobiomeBestPracticeReview repository. Record which paradigm was used (phase, data type = amplicon, tool, role) so downstream comparative/statistical analysis is interpretable.

## Related tools

- **mothur** (OTU-based amplicon workflow: clusters 16S rRNA reads into operational taxonomic units at the 97% identity threshold using reference taxonomy databases (Silva nr_v132, Greengenes, RDP)) — https://github.com/grimmlab/MicrobiomeBestPracticeReview
- **DADA2** (ASV-based amplicon workflow: applies a denoising approach on biological sequences to infer exact amplicon sequence variants instead of clustering) — https://github.com/grimmlab/MicrobiomeBestPracticeReview
- **phyloseq** (Standard visualization of processed OTU/ASV amplicon data in the best-practice workflow)

## Examples

```
sh amplicon_analysis.sh
```

## Evaluation signals

- The chosen paradigm is explicitly documented: OTU clustering at the 97% identity threshold versus ASV denoising — not an undocumented hybrid.
- OTU outputs are clustered into operational taxonomic units at the commonly used 97% identity threshold, matching the review's stated cutoff.
- ASV outputs contain exact (denoised) amplicon sequence variants rather than 97%-pooled clusters, and no clustering step was silently applied on top of DADA2.
- A reference taxonomy database appropriate to the classifier was used and recorded (silva.nr_v132, Gg_13_8_99, or Trainset16_022016.pds).
- The result table includes one row per amplicon method (mothur OTU workflow and DADA2 ASV workflow) with columns phase, data type = amplicon, tool, and role, and both are represented in best_practice_tools_map.csv.

## Limitations

- OTU-based clustering depends on a predefined identity threshold (commonly 97%), so closely related strains can be merged or split arbitrarily; ASV denoising avoids threshold effects but is a different unit of measurement and results are not directly comparable between the two paradigms.
- Classification accuracy depends on the chosen reference taxonomy database (Silva, Greengenes, RDP), which must be downloaded and kept version-consistent (e.g., nr_v132).
- The review's workflows were tested on V4 16S rRNA data on Ubuntu 18.04.2 with specific hardware (20-core CPU, 128 GB RAM); other hypervariable regions or platforms are outside the validated scope.

## Evidence

- [methods] OTU clustering vs ASV denoising distinction: "OTU based methods cluster the reads based on a predefined identity threshold (commonly 97%) into operational taxonomic units"
- [methods] ASV approach characterization: "ASV based tool utilizes a denoising approach on biological sequences in samples"
- [readme] Two amplicon workflows provided (mothur and dada2): "First workflow for amplicon, using the standard mothur and dada2, and along with it some standard visualization are provided for the processed data."
- [methods] Reference taxonomy databases used by the mothur workflow: "https://mothur.org/w/images/3/32/Silva.nr_v132.tgz https://zenodo.org/record/1172783/files/silva_nr_v132_train_set.fa.gz"
- [readme] Amplicon workflow is script-driven: "Both the workflows are controlled by bash scripts: `amplicon_analysis.sh` and `metagenomics_analysis.sh`."
