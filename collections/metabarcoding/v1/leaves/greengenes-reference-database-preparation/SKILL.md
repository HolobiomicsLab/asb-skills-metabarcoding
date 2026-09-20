---
name: greengenes-reference-database-preparation
description: Use when the EasyAmplicon protocol requires QIIME 2-based species annotation
  and no pre-trained Greengenes classifier exists, so the 99% OTU sequences and taxonomy
  must be downloaded, imported, and trained into classifier_gg_13_8_99.qza.
when_to_use_negative:
- A pre-trained classifier_gg_13_8_99.qza already exists and loads as a valid TaxonomicClassifier
  artifact — retraining wastes ~30 min.
- The target analysis is USEARCH/VSEARCH-based taxonomic assignment with RDP (rdp_16s_v18)
  rather than QIIME 2 species annotation.
- ITS/fungal analysis is the goal — UNITE, not Greengenes (16S), is the appropriate
  database.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Imports reference reads/taxonomy and trains the naive-Bayes taxonomic classifier
    via qiime feature-classifier fit-classifier-naive-bayes
- name: EasyAmplicon
  role: Protocol in which the classifier training step is embedded and the result
    directory conventions are defined
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: Greengenes
  role: Source reference database (gg_13_8_otus archive, 99% OTU sequences and taxonomy)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:dbcf9417c4b51029a75bc71be09e9d1aeebd7d6c0bf17d3e7cf356deada8e047
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - Greengenes
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# greengenes-reference-database-preparation

## Summary

Prepares the Greengenes 13.8 99%-identity reference database for QIIME 2 species-level taxonomic annotation by importing 99_otus.fasta and its taxonomy into FeatureData artifacts and training a naive-Bayes classifier (classifier_gg_13_8_99.qza).

## When to use

Use when the EasyAmplicon protocol requires QIIME 2-based species annotation and no pre-trained Greengenes classifier exists, so the 99% OTU sequences and taxonomy must be downloaded, imported, and trained into classifier_gg_13_8_99.qza.

## When NOT to use

- A pre-trained classifier_gg_13_8_99.qza already exists and loads as a valid TaxonomicClassifier artifact — retraining wastes ~30 min.
- The target analysis is USEARCH/VSEARCH-based taxonomic assignment with RDP (rdp_16s_v18) rather than QIIME 2 species annotation.
- ITS/fungal analysis is the goal — UNITE, not Greengenes (16S), is the appropriate database.

## Inputs

- Greengenes gg_13_8_otus archive (99_otus.fasta, 99% OTU reference sequences)
- Greengenes 99% OTU taxonomy file
- QIIME 2 environment (Conda yml, e.g. q2-py39-linux-conda.yml)

## Outputs

- 99_otus.qza (FeatureData[Sequence] artifact)
- ref-taxonomy.qza (FeatureData[Taxonomy] artifact)
- classifier_gg_13_8_99.qza (TaxonomicClassifier artifact)

## How to apply

Download the gg_13_8_otus archive from the Greengenes release site (https://ftp.microbio.me/greengenes_release/current/) and extract the 99%-identity reference sequences (99_otus.fasta) plus its taxonomy file. Import 99_otus.fasta with `qiime tools import` as a FeatureData[Sequence] artifact (99_otus.qza) and the taxonomy file as a FeatureData[Taxonomy] artifact (ref-taxonomy.qza). Train the classifier with `qiime feature-classifier fit-classifier-naive-bayes`, supplying the imported reference reads and reference taxonomy as inputs, which may take about 30 minutes. Save the result as classifier_gg_13_8_99.qza in the result directory; within EasyAmplicon this typically runs inside a QIIME 2 Conda environment created in WSL (e.g. `conda env create -n ${n} --file ${n}-py39-linux-conda.yml`).

## Related tools

- **QIIME 2** (Imports reference reads/taxonomy and trains the naive-Bayes taxonomic classifier via qiime feature-classifier fit-classifier-naive-bayes)
- **EasyAmplicon** (Protocol in which the classifier training step is embedded and the result directory conventions are defined) — https://github.com/YongxinLiu/EasyAmplicon
- **Greengenes** (Source reference database (gg_13_8_otus archive, 99% OTU sequences and taxonomy))

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads 99_otus.qza --i-reference-taxonomy ref-taxonomy.qza --o-classifier classifier_gg_13_8_99.qza
```

## Evaluation signals

- classifier_gg_13_8_99.qza exists in the result directory.
- classifier_gg_13_8_99.qza loads as a valid QIIME 2 artifact with semantic type TaxonomicClassifier.
- 99_otus.qza validates as FeatureData[Sequence] and ref-taxonomy.qza validates as FeatureData[Taxonomy] after `qiime tools import`.
- The fit-classifier-naive-bayes run completes without error (expected runtime ~30 min).

## Limitations

- Classifier training with fit-classifier-naive-bayes may take up to 30 minutes.
- The paper's database section highlights Greengenes 97% OTUs (e.g., for PICRUSt) as the pre-packaged form; the 99% OTUs needed for species-level classifier training must be downloaded separately from the Greengenes release site.

## Evidence

- [intro] Greengenes download in USEARCH-compatible format: "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
- [intro] QIIME 2 Conda environment setup in EasyAmplicon: [evidence span withheld — anchor status: not_found]
- [methods] Pre-packaged reference databases including Greengenes OTUs: "Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt)"
- [readme] Official EasyAmplicon repository: "We highly recommend downloading them from the official GitHub repository at https://github.com/YongxinLiu/EasyAmplicon."
