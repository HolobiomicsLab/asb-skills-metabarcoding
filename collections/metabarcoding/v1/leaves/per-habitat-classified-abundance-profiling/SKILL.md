---
name: per-habitat-classified-abundance-profiling
description: Use when you need per-sample, species-level classified taxonomic abundance
  profiles for a defined environment/habitat (e.g., an EMPO 3 category like human-stool
  or soil-non-saline) to build bespoke class weights.
when_to_use_negative:
- The sample type lacks sufficient characterization in Qiita/redbiom to assemble bespoke
  weights — use average weights estimated from global microbial species distributions
  instead
- The input is already a classified abundance table or pre-built weights (e.g., readytowear
  ready-made weights for a common primer/database/habitat combination)
- The sample distribution is highly unusual (e.g., synthetic populations) where weights
  must be compiled manually from existing samples rather than from a standard habitat
  query
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: q2-clawback
  role: QIIME 2 plugin whose assemble_weights_from_Qiita pipeline performs the metadata
    summarization, sample fetching, variant extraction, classification, and per-habitat
    abundance assembly
  repo: github.com/BenKaehler/q2-clawback
- name: redbiom
  role: Cache service used to summarize Qiita metadata categories/contexts and fetch
    matching samples and their sequence variants for the target habitat
  repo: https://github.com/biocore/redbiom
- name: q2-feature-classifier
  role: Provides the scikit-learn naive Bayes classifier used to force species-level
    (confidence -1) classification of extracted variants under uniform class weights
- name: QIIME 2
  role: Framework in which the q2-clawback plugin actions and the classifier are run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:50c14306315bdb5b132ef86c0e4e4b31e75fc102f8a3fd0708b082a611512e05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# per-habitat-classified-abundance-profiling

## Summary

Assembles species-level classified taxonomic abundances for samples from a target habitat (e.g., one of 14 EMPO 3 habitats spanning 21,513 Qiita samples) so that downstream generate_class_weights can produce environment-specific taxonomic weights for q2-feature-classifier's naive Bayes classifier.

## When to use

Use when you need per-sample, species-level classified taxonomic abundance profiles for a defined environment/habitat (e.g., an EMPO 3 category like human-stool or soil-non-saline) to build bespoke class weights.

## When NOT to use

- The sample type lacks sufficient characterization in Qiita/redbiom to assemble bespoke weights — use average weights estimated from global microbial species distributions instead
- The input is already a classified abundance table or pre-built weights (e.g., readytowear ready-made weights for a common primer/database/habitat combination)
- The sample distribution is highly unusual (e.g., synthetic populations) where weights must be compiled manually from existing samples rather than from a standard habitat query

## Inputs

- Qiita metadata category name (e.g., EMPO 3 habitat) identifying the target sample environment
- redbiom context (processing type, e.g., Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4) for feature retrieval
- Feature reference (sequence variants / feature IDs) from fetched Qiita samples retrieved via redbiom
- q2-feature-classifier naive Bayes classifier trained on the reference database (e.g., Greengenes or SILVA 138.1)

## Outputs

- classified_abundances.csv containing species-level (confidence -1) classified taxonomic abundances per sample for the habitat
- Habitat-specific taxonomic weight profiles consumed by generate_class_weights

## How to apply

Enumerate the Qiita metadata category and its available processing contexts for the target habitat using summarize_Qiita_metadata_category_and_contexts, then fetch the matching samples and their sequence variants through redbiom with fetch_Qiita_samples. Extract sequence variants from the fetched samples using sequence_variants_from_samples. Classify the extracted variants with a q2-feature-classifier scikit-learn naive Bayes classifier under uniform class weights, forcing classification to the species level by setting confidence to -1 (no rejection). Assemble the per-sample classified taxonomic abundances for the habitat and write them to classified_abundances.csv, which downstream generate_class_weights consumes. The rationale is that naive Bayes classifiers assume all reference species are equally likely to be observed, and this assumption is violated in practice; empirical per-habitat abundances supply the prior information needed for accurate species-level classification.

## Related tools

- **q2-clawback** (QIIME 2 plugin whose assemble_weights_from_Qiita pipeline performs the metadata summarization, sample fetching, variant extraction, classification, and per-habitat abundance assembly) — github.com/BenKaehler/q2-clawback
- **redbiom** (Cache service used to summarize Qiita metadata categories/contexts and fetch matching samples and their sequence variants for the target habitat) — https://github.com/biocore/redbiom
- **q2-feature-classifier** (Provides the scikit-learn naive Bayes classifier used to force species-level (confidence -1) classification of extracted variants under uniform class weights)
- **QIIME 2** (Framework in which the q2-clawback plugin actions and the classifier are run)

## Examples

```
redbiom search metadata 'EMPO_3' | redbiom fetch samples --context $ctx --output habitat.biom
```

## Evaluation signals

- classified_abundances.csv contains species-level classifications (confidence = -1, no rejection) for every fetched sample of the habitat
- Number of samples profiled matches the count of Qiita samples returned by fetch_Qiita_samples for the chosen metadata category and context
- Coverage aligns with the article's scale: 14 EMPO 3 habitats across 21,513 samples, with species-level classification improvement demonstrated with as few as 122 samples (four-fifths of saline surface EMP samples)
- Resulting weights, when passed to generate_class_weights and used in classification, reduce species-level error rate below the uniform-weight baseline (25% toward 14%)
- Taxonomic labels in the abundance table are consistent with the reference database taxonomy used to train the naive Bayes classifier

## Limitations

- Performance degradation was observed for rare species under bespoke weighting
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into the taxonomic weights
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)
- Systematic errors from bias in common sequencing techniques are not controlled without integrating multiple technologies for microbial distribution estimation

## Evidence

- [methods] q2-clawback assembles weights from those samples, producing weights for 14 EMPO 3 habitats across 21,513 samples.: [evidence span withheld — anchor status: not_found]
- [methods] Classify the extracted variants with a q2-feature-classifier naive Bayes classifier (scikit-learn) under uniform class weights, forcing classification to the species level by setting confidence to -1.: [evidence span withheld — anchor status: not_found]
- [methods] The weight-assembly mechanism works by querying Qiita on an available metadata category (e.g., EMPO 3 habitat) or any curated sample set: [evidence span withheld — anchor status: not_found]
- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [discussion] We demonstrated species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study; Table 1): "We demonstrated species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study; Table 1)"
- [readme] Redbiom is a cache service for sample metadata and sample data. It allows for rapidly: finding samples by the features they contain: "Redbiom is a cache service for sample metadata and sample data. It allows for rapidly:

* finding samples by the features they contain"
