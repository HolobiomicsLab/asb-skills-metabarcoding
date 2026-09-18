---
name: naive-bayes-taxonomic-classifier-training
description: Use when a QIIME 2-based species-annotation step needs a pre-trained
  classifier (e.g., classifier_gg_13_8_99.qza) and only the raw Greengenes 99_otus.fasta
  and its taxonomy file are available, requiring fit-classifier-naive-bayes training.
when_to_use_negative:
- A pre-trained classifier artifact (e.g., classifier_gg_13_8_99.qza) already exists
  — reuse it instead of retraining
- The analysis pipeline is USEARCH/VSEARCH-based (EasyAmplicon default route), which
  uses USEARCH-compatible reference databases (e.g., RDP rdp_16s_v18) rather than
  QIIME 2 classifiers
- The target reference is not 16S bacterial (e.g., ITS fungi/eukaryotes) — the UNITE
  database is the appropriate reference for ITS analysis
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3307
tools:
- name: QIIME 2
  role: Provides `qiime tools import` for building FeatureData[Sequence]/FeatureData[Taxonomy]
    artifacts and `qiime feature-classifier fit-classifier-naive-bayes` for training
    the TaxonomicClassifier
  repo: https://qiime2.org
- name: EasyAmplicon
  role: Protocol that packages and documents the classifier-training step (downloading
    Greengenes, importing artifacts, running fit-classifier-naive-bayes) as an optional
    QIIME 2 route
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Companion database/software directory of the EasyAmplicon pipeline that hosts
    reference databases and scripts used alongside the QIIME 2 annotation route
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:114e8d5c7b120eb451e3dd5ced50400bb44eba1796675e961210b5f582eb2395
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# naive-bayes-taxonomic-classifier-training

## Summary

Trains a QIIME 2 naive-Bayes taxonomic classifier by importing Greengenes 99% OTU reference sequences and taxonomy as QIIME 2 artifacts and running `qiime feature-classifier fit-classifier-naive-bayes`, producing a reusable TaxonomicClassifier artifact (classifier_gg_13_8_99.qza) for species-level annotation of amplicon data within the EasyAmplicon protocol.

## When to use

Use when a QIIME 2-based species-annotation step needs a pre-trained classifier (e.g., classifier_gg_13_8_99.qza) and only the raw Greengenes 99_otus.fasta and its taxonomy file are available, requiring fit-classifier-naive-bayes training.

## When NOT to use

- A pre-trained classifier artifact (e.g., classifier_gg_13_8_99.qza) already exists — reuse it instead of retraining
- The analysis pipeline is USEARCH/VSEARCH-based (EasyAmplicon default route), which uses USEARCH-compatible reference databases (e.g., RDP rdp_16s_v18) rather than QIIME 2 classifiers
- The target reference is not 16S bacterial (e.g., ITS fungi/eukaryotes) — the UNITE database is the appropriate reference for ITS analysis

## Inputs

- Greengenes 99%-identity OTU reference sequences (99_otus.fasta), downloaded from the Greengenes release site
- Greengenes OTU taxonomy file
- QIIME 2 environment (Conda env) installed per the EasyAmplicon protocol

## Outputs

- classifier_gg_13_8_99.qza — a QIIME 2 artifact of semantic type TaxonomicClassifier

## How to apply

Download the gg_13_8_otus archive from the Greengenes release site and extract the 99%-identity reference sequences (99_otus.fasta) and its taxonomy file. Import 99_otus.fasta with `qiime tools import` as a FeatureData[Sequence] artifact (99_otus.qza) and the taxonomy file as a FeatureData[Taxonomy] artifact (ref-taxonomy.qza). Train the classifier with `qiime feature-classifier fit-classifier-naive-bayes`, supplying the imported reference reads and reference taxonomy as inputs, and save the result as classifier_gg_13_8_99.qza in the result directory. Training may take about 30 minutes, so run it inside the QIIME 2 Conda environment that EasyAmplicon installs (e.g., created with `conda env create -n ${n} --file ${n}-py39-linux-conda.yml`).

## Related tools

- **QIIME 2** (Provides `qiime tools import` for building FeatureData[Sequence]/FeatureData[Taxonomy] artifacts and `qiime feature-classifier fit-classifier-naive-bayes` for training the TaxonomicClassifier) — https://qiime2.org
- **EasyAmplicon** (Protocol that packages and documents the classifier-training step (downloading Greengenes, importing artifacts, running fit-classifier-naive-bayes) as an optional QIIME 2 route) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Companion database/software directory of the EasyAmplicon pipeline that hosts reference databases and scripts used alongside the QIIME 2 annotation route) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-ref-reads 99_otus.qza --i-ref-taxonomy ref-taxonomy.qza --o-classifier classifier_gg_13_8_99.qza
```

## Evaluation signals

- classifier_gg_13_8_99.qza exists in the result directory after training
- classifier_gg_13_8_99.qza loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier
- The imported 99_otus.qza artifact is of type FeatureData[Sequence] and ref-taxonomy.qza is of type FeatureData[Taxonomy]
- fit-classifier-naive-bayes completes without error (runtime may be ~30 min on Greengenes 99% references)

## Limitations

- Classifier training on the Greengenes 99% OTU set may take up to 30 minutes, so sufficient compute time is needed
- Requires a working QIIME 2 Conda environment (EasyAmplicon installs it via Conda, e.g., in WSL on Windows), adding an environment dependency to the otherwise USEARCH/VSEARCH-centric pipeline
- Training uses Greengenes 99% OTUs; the article also notes Greengenes 97% OTUs are packaged for PICRUSt, so reference-identity choice must match the downstream task

## Evidence

- [methods] Greengenes 99% OTU classifier training: [evidence span withheld — anchor status: not_found]
- [intro] QIIME 2 platform description: "QIIME 2: A next‐generation microbiome bioinformatics platform that is extensible, free, open source, and community‐developed"
- [intro] QIIME 2 environment creation step: [evidence span withheld — anchor status: not_found]
- [intro] Greengenes database availability: "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
