---
name: qiime2-artifact-import
description: Use when downloaded reference sequences (FASTA) or taxonomy files must
  be converted into typed QIIME 2 artifacts (FeatureData[Sequence], FeatureData[Taxonomy])
  before classifier training.
when_to_use_negative:
- The data is already a QIIME 2 artifact (.qza) — no re-import is needed and re-importing
  would fail type validation
- You only need USEARCH/VSEARCH-compatible reference databases (e.g., RDP rdp_16s_v18.fa
  for chimera checking), which are used directly without QIIME artifact conversion
- Importing raw amplicon reads for the main EasyAmplicon OTU/ASV workflow, which uses
  USEARCH/VSEARCH commands rather than QIIME 2
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: QIIME 2
  role: Provides `qiime tools import` for creating typed artifacts (FeatureData[Sequence],
    FeatureData[Taxonomy]) and the feature-classifier plugin for downstream naive-Bayes
    training
  repo: https://qiime2.org
- name: EasyAmplicon
  role: Protocol that embeds the QIIME 2 Conda environment setup and Greengenes classifier-training
    workflow
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:49d05257c633120ac2bc5d1de9a41b2c5a90b76010deab2146e9b1b4e2015146
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EasyAmplicon
  - QIIME 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# qiime2-artifact-import

## Summary

Converts raw reference files (e.g., Greengenes 99% OTU FASTA sequences and taxonomy files) into typed QIIME 2 artifacts using `qiime tools import`, a prerequisite step in the EasyAmplicon protocol for training a naive-Bayes species classifier.

## When to use

Use when downloaded reference sequences (FASTA) or taxonomy files must be converted into typed QIIME 2 artifacts (FeatureData[Sequence], FeatureData[Taxonomy]) before classifier training.

## When NOT to use

- The data is already a QIIME 2 artifact (.qza) — no re-import is needed and re-importing would fail type validation
- You only need USEARCH/VSEARCH-compatible reference databases (e.g., RDP rdp_16s_v18.fa for chimera checking), which are used directly without QIIME artifact conversion
- Importing raw amplicon reads for the main EasyAmplicon OTU/ASV workflow, which uses USEARCH/VSEARCH commands rather than QIIME 2

## Inputs

- 99_otus.fasta (Greengenes 99%-identity reference sequences)
- Greengenes taxonomy file (e.g., 99_otu_taxonomy.txt)

## Outputs

- 99_otus.qza (QIIME 2 artifact of type FeatureData[Sequence])
- ref-taxonomy.qza (QIIME 2 artifact of type FeatureData[Taxonomy])

## How to apply

Download the gg_13_8_otus archive from the Greengenes release site and extract the 99%-identity reference sequences (99_otus.fasta) plus its taxonomy file. Import 99_otus.fasta with `qiime tools import` specifying semantic type FeatureData[Sequence] to produce 99_otus.qza, and import the taxonomy file as FeatureData[Taxonomy] to produce ref-taxonomy.qza. These typed artifacts are then supplied as inputs to `qiime feature-classifier fit-classifier-naive-bayes`, which trains the classifier (classifier_gg_13_8_99.qza, ~30 min runtime). The QIIME 2 environment itself is set up under WSL via Conda (`conda env create -n ${n} --file ${n}-py39-linux-conda.yml`). Correct semantic typing at import is essential because QIIME 2 validates artifact content against the declared type.

## Related tools

- **QIIME 2** (Provides `qiime tools import` for creating typed artifacts (FeatureData[Sequence], FeatureData[Taxonomy]) and the feature-classifier plugin for downstream naive-Bayes training) — https://qiime2.org
- **EasyAmplicon** (Protocol that embeds the QIIME 2 Conda environment setup and Greengenes classifier-training workflow) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
qiime tools import --input-path 99_otus.fasta --type 'FeatureData[Sequence]' --output-path 99_otus.qza; qiime tools import --input-path ref-taxonomy.txt --type 'FeatureData[Taxonomy]' --output-path ref-taxonomy.qza
```

## Evaluation signals

- 99_otus.qza exists and loads as a valid QIIME 2 artifact with semantic type FeatureData[Sequence]
- ref-taxonomy.qza exists and loads as a valid QIIME 2 artifact with semantic type FeatureData[Taxonomy]
- The downstream classifier_gg_13_8_99.qza is produced within the expected ~30 min runtime and validates as semantic type TaxonomicClassifier
- `qiime tools peek` on each artifact reports the expected semantic type matching the declared import type

## Limitations

- Naive-Bayes classifier training is computationally expensive — the protocol notes it may take about 30 minutes
- The pipeline's QIIME 2 environment is provisioned via Conda under WSL, so Windows users must operate through the WSL layer
- The article index shows Greengenes 97% OTUs pre-packaged for PICRUSt and RDP for chimera checking, so the 99% Greengenes import path is specific to QIIME 2 species-annotation classifier training, not the default EasyAmplicon taxonomic assignment

## Evidence

- [methods] Import 99_otus.fasta into a QIIME 2 artifact of type FeatureData[Sequence] and the taxonomy file into a FeatureData[Taxonomy] artifact using qiime tools import.: [evidence span withheld — anchor status: not_found]
- [methods] qiime feature-classifier fit-classifier-naive-bayes to output classifier_gg_13_8_99.qza, which may take 30 min.: [evidence span withheld — anchor status: not_found]
- [intro] conda env create ‐n ${n} ‐‐file ${n}‐py39‐linux‐conda.yml: [evidence span withheld — anchor status: not_found]
- [intro] QIIME 2: A next‐generation microbiome bioinformatics platform that is extensible, free, open source, and community‐developed: "QIIME 2: A next‐generation microbiome bioinformatics platform that is extensible, free, open source, and community‐developed (https://qiime2.org)."
