---
name: amplicon-reference-sequence-handling
description: Use when a QIIME 2 amplicon pipeline needs a species-annotation classifier
  and a reference database such as Greengenes 13_8 99% OTUs must be downloaded, imported,
  and trained into a TaxonomicClassifier artifact.
when_to_use_negative:
- Taxonomy can be assigned with a pre-packaged database already in the EasyMicrobiome
  folder (e.g., RDP rdp_16s_v18.fa for chimera/reference checks or SILVA) without
  needing a trained QIIME 2 classifier
- The analysis runs entirely in the USEARCH/VSEARCH branch of EasyAmplicon, which
  consumes USEARCH-compatible Greengenes files directly rather than QIIME 2 artifacts
- A trained classifier artifact is already provided and only needs to be passed to
  classify
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Imports the Greengenes reference sequences and taxonomy as artifacts and runs
    feature-classifier fit-classifier-naive-bayes to train the classifier
- name: EasyAmplicon
  role: Protocol that packages this classifier-training workflow (via a QIIME 2 Conda
    environment in WSL) and its result directory conventions
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: EasyMicrobiome
  role: Companion database directory providing pre-packaged reference databases (RDP/SILVA/UNITE,
    Greengenes 97% OTUs) as alternatives
  repo: https://github.com/YongxinLiu/EasyMicrobiome
- name: Greengenes
  role: Source reference database (gg_13_8_otus, 99% OTUs) downloaded from the Greengenes
    release site
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:bdd6cc5b42a9164d9ad72431e399ece3ec3cfc58682e502a513b703587d38ea3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/EasyAmplicon
  tools:
  - EasyAmplicon
  - EasyMicrobiome
  - Greengenes
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# amplicon-reference-sequence-handling

## Summary

Downloads a Greengenes reference database, imports its 99%-OTU sequences and taxonomy into QIIME 2 artifacts, and trains a naive-Bayes taxonomic classifier (classifier_gg_13_8_99.qza) for species-level annotation in the EasyAmplicon protocol.

## When to use

Use when a QIIME 2 amplicon pipeline needs a species-annotation classifier and a reference database such as Greengenes 13_8 99% OTUs must be downloaded, imported, and trained into a TaxonomicClassifier artifact.

## When NOT to use

- Taxonomy can be assigned with a pre-packaged database already in the EasyMicrobiome folder (e.g., RDP rdp_16s_v18.fa for chimera/reference checks or SILVA) without needing a trained QIIME 2 classifier
- The analysis runs entirely in the USEARCH/VSEARCH branch of EasyAmplicon, which consumes USEARCH-compatible Greengenes files directly rather than QIIME 2 artifacts
- A trained classifier artifact is already provided and only needs to be passed to classify

## Inputs

- Greengenes gg_13_8_otus archive with 99%-identity OTU representative sequences (99_otus.fasta)
- Greengenes taxonomy file corresponding to the 99% OTUs

## Outputs

- FeatureData[Sequence] QIIME 2 artifact (99_otus.qza)
- FeatureData[Taxonomy] QIIME 2 artifact (ref-taxonomy.qza)
- TaxonomicClassifier QIIME 2 artifact (classifier_gg_13_8_99.qza)

## How to apply

Download the gg_13_8_otus archive from the Greengenes release site (https://ftp.microbio.me/greengenes_release/current/) and extract the 99%-identity reference sequences (99_otus.fasta) and its taxonomy file. Import the FASTA into a QIIME 2 artifact of semantic type FeatureData[Sequence] and the taxonomy file into a FeatureData[Taxonomy] artifact using `qiime tools import`. Train the classifier with `qiime feature-classifier fit-classifier-naive-bayes`, supplying the imported reference reads and reference taxonomy as inputs, and save the output as classifier_gg_13_8_99.qza in the result directory; the fit step may take about 30 minutes. Validate that classifier_gg_13_8_99.qza exists and loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier. Note the article also distributes pre-packaged references (RDP/SILVA/UNITE for taxonomy) via the EasyMicrobiome database folder when Greengenes is not the target reference.

## Related tools

- **QIIME 2** (Imports the Greengenes reference sequences and taxonomy as artifacts and runs feature-classifier fit-classifier-naive-bayes to train the classifier)
- **EasyAmplicon** (Protocol that packages this classifier-training workflow (via a QIIME 2 Conda environment in WSL) and its result directory conventions) — https://github.com/YongxinLiu/EasyAmplicon
- **EasyMicrobiome** (Companion database directory providing pre-packaged reference databases (RDP/SILVA/UNITE, Greengenes 97% OTUs) as alternatives) — https://github.com/YongxinLiu/EasyMicrobiome
- **Greengenes** (Source reference database (gg_13_8_otus, 99% OTUs) downloaded from the Greengenes release site)

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads 99_otus.qza --i-reference-taxonomy ref-taxonomy.qza --o-classifier classifier_gg_13_8_99.qza
```

## Evaluation signals

- classifier_gg_13_8_99.qza exists in the result directory
- classifier_gg_13_8_99.qza loads as a valid QIIME 2 artifact of semantic type TaxonomicClassifier
- The imported reference artifacts carry the correct semantic types: FeatureData[Sequence] for 99_otus.qza and FeatureData[Taxonomy] for ref-taxonomy.qza
- The fit-classifier-naive-bayes run completes without error (may take ~30 min), producing a non-empty classifier artifact

## Limitations

- Training fit-classifier-naive-bayes may take around 30 minutes, so the step should be planned for accordingly
- The Greengenes download instructions in the article emphasize the USEARCH-compatible format, so the correct 99% OTU FASTA and taxonomy files must be selected and separately imported for the QIIME 2 route
- EasyAmplicon's QIIME 2 environment is installed via Conda inside WSL (conda env create -n ${n} --file ${n}-py39-linux-conda.yml), which is a prerequisite for this skill

## Evidence

- [other] Finding from source card: [evidence span withheld — anchor status: not_found]
- [intro] Greengenes download location: "users can download Greengenes database (https://ftp.microbio.me/greengenes_release/current/) in USEARCH‐compatible format"
- [methods] Pre-packaged reference databases: "Pre-packaged reference databases (e.g., RDP/SILVA/UNITE for taxonomy, Greengenes 97% OTUs for PICRUSt)"
- [intro] QIIME 2 environment setup prerequisite: [evidence span withheld — anchor status: not_found]
- [other] Validation requirement: [evidence span withheld — anchor status: not_found]
