---
name: qiime2-semantic-type-validation
description: Use when a QIIME 2 step in the EasyAmplicon pipeline has just produced
  or consumed an artifact (.qza) and you must confirm it loads and has the semantic
  type expected by the next command.
when_to_use_negative:
- Input is raw FASTQ reads or a plain FASTA/taxonomy file that has not yet been imported
  into a .qza artifact — import first, then validate
- The analysis path uses the USEARCH/VSEARCH branch of EasyAmplicon (e.g., vsearch
  --uchime_ref, usearch -otutab_trim) where no QIIME 2 artifacts exist
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: QIIME 2
  role: Provides `qiime tools import` and `qiime feature-classifier fit-classifier-naive-bayes`;
    its artifact/semantic-type system (FeatureData[Sequence], FeatureData[Taxonomy],
    TaxonomicClassifier) is what is validated
- name: EasyAmplicon
  role: Protocol embedding the QIIME 2 Greengenes classifier-training steps whose
    artifacts are validated
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Accompanying scripts and databases directory installed alongside EasyAmplicon
  repo: https://github.com/YongxinLiu/EasyMicrobiome
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:9435b566f5d3af8a4e7e77f9ffff06e515e80ec9b0cb6dadcc3551f2c8fd4e94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# qiime2-semantic-type-validation

## Summary

Validates that QIIME 2 artifacts produced in the EasyAmplicon protocol (e.g., imported Greengenes references and the trained naive-Bayes classifier) exist, load correctly, and carry the expected semantic types such as FeatureData[Sequence], FeatureData[Taxonomy], and TaxonomicClassifier.

## When to use

Use when a QIIME 2 step in the EasyAmplicon pipeline has just produced or consumed an artifact (.qza) and you must confirm it loads and has the semantic type expected by the next command.

## When NOT to use

- Input is raw FASTQ reads or a plain FASTA/taxonomy file that has not yet been imported into a .qza artifact — import first, then validate
- The analysis path uses the USEARCH/VSEARCH branch of EasyAmplicon (e.g., vsearch --uchime_ref, usearch -otutab_trim) where no QIIME 2 artifacts exist

## Inputs

- 99_otus.fasta (Greengenes 99% OTU reference sequences)
- Greengenes taxonomy file
- QIIME 2 artifacts (.qza): 99_otus.qza (FeatureData[Sequence]), ref-taxonomy.qza (FeatureData[Taxonomy]), classifier_gg_13_8_99.qza (TaxonomicClassifier)

## Outputs

- Confirmation that classifier_gg_13_8_99.qza exists and loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier
- Reported semantic types of imported artifacts matching the expected FeatureData types

## How to apply

After each QIIME 2 step in the EasyAmplicon protocol, check the artifact before proceeding. When importing the Greengenes 99%-identity reference sequences (99_otus.fasta) and its taxonomy file, use `qiime tools import` with explicit types FeatureData[Sequence] and FeatureData[Taxonomy]. After training with `qiime feature-classifier fit-classifier-naive-bayes` (which may take ~30 min), confirm that classifier_gg_13_8_99.qza exists in the result directory and loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier. This gating prevents downstream taxonomic-annotation commands from failing on mistyped or corrupt artifacts.

## Related tools

- **QIIME 2** (Provides `qiime tools import` and `qiime feature-classifier fit-classifier-naive-bayes`; its artifact/semantic-type system (FeatureData[Sequence], FeatureData[Taxonomy], TaxonomicClassifier) is what is validated)
- **EasyAmplicon** (Protocol embedding the QIIME 2 Greengenes classifier-training steps whose artifacts are validated) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Accompanying scripts and databases directory installed alongside EasyAmplicon) — https://github.com/YongxinLiu/EasyMicrobiome

## Examples

```
qiime tools import --input-path 99_otus.fasta --type 'FeatureData[Sequence]' --output-path 99_otus.qza
```

## Evaluation signals

- classifier_gg_13_8_99.qza exists in the result directory and loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier
- 99_otus.qza reports semantic type FeatureData[Sequence] after import of 99_otus.fasta
- ref-taxonomy.qza reports semantic type FeatureData[Taxonomy] after import of the Greengenes taxonomy file
- The QIIME 2 Conda environment (created from ${n}-py39-linux-conda.yml) resolves all commands without plugin errors

## Limitations

- Classifier training with fit-classifier-naive-bayes may take about 30 minutes, so validation must be performed only after the command completes
- The protocol relies on a Conda-installed QIIME 2 environment (e.g., inside WSL); semantic-type validation is meaningless if the wrong environment or QIIME 2 version is activated

## Evidence

- [methods] Import 99_otus.fasta into a QIIME 2 artifact of type FeatureData[Sequence] and the taxonomy file into a FeatureData[Taxonomy] artifact using qiime tools import.: [evidence span withheld — anchor status: not_found]
- [methods] Validation: verify that classifier_gg_13_8_99.qza exists and loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier.: [evidence span withheld — anchor status: not_found]
- [intro] install Conda in WSL and create a QIIME 2 Conda environment: "conda env create ‐n ${n} ‐‐file ${n}‐py39‐linux‐conda.yml"
