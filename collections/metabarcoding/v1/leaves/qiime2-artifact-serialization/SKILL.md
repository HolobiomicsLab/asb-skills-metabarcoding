---
name: qiime2-artifact-serialization
description: Use when habitat-specific taxonomic weight vectors computed by q2-clawback's
  generate_class_weights must be persisted as QIIME 2 FeatureWeights artifacts (class_weights.qza)
  or exchanged for downstream classifier training.
when_to_use_negative:
- When ready-made weights already exist for your primer, reference database, and habitat
  combination (e.g., animal-distal-gut, soil-non-saline, human-stool EMPO 3 types
  on Zenodo via readytowear) — download instead of regenerating and re-serializing.
- When the goal is the classification itself rather than weight assembly — serialization
  is an intermediate step, not the classifier-fitting step of q2-feature-classifier.
- When the input is an arbitrary per-taxon count table not derived from classified
  sequence variants per habitat — the FeatureWeights semantics assume clawback's aggregated,
  normalized, offset vectors.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: q2-clawback
  role: plugin whose generate_class_weights action produces the weight vectors that
    get serialized as FeatureWeights artifacts
  repo: https://github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: framework providing the artifact (.qza) format and FeatureWeights semantic
    type used for serialization and later loading
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: downstream consumer that applies the serialized FeatureWeights to train habitat-specific
    naive Bayes classifiers
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: readytowear
  role: repository of ready-made clawback-generated taxonomic weights; shares the
    same serialized FeatureWeights artifact contract
  repo: https://github.com/BenKaehler/readytowear
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:b5bf731d77870a694f94823b3861197587176786a461a8efe241bb60f6fc26e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# qiime2-artifact-serialization

## Summary

Serializes q2-clawback's per-habitat aggregated, normalized, and offset taxonomic weight vectors as QIIME 2 artifacts of semantic type FeatureWeights (.qza) so environment-specific weights can be consumed by q2-feature-classifier and shared with the readytowear inventory.

## When to use

Use when habitat-specific taxonomic weight vectors computed by q2-clawback's generate_class_weights must be persisted as QIIME 2 FeatureWeights artifacts (class_weights.qza) or exchanged for downstream classifier training.

## When NOT to use

- When ready-made weights already exist for your primer, reference database, and habitat combination (e.g., animal-distal-gut, soil-non-saline, human-stool EMPO 3 types on Zenodo via readytowear) — download instead of regenerating and re-serializing.
- When the goal is the classification itself rather than weight assembly — serialization is an intermediate step, not the classifier-fitting step of q2-feature-classifier.
- When the input is an arbitrary per-taxon count table not derived from classified sequence variants per habitat — the FeatureWeights semantics assume clawback's aggregated, normalized, offset vectors.

## Inputs

- per-sample classified sequence-variant counts (output of sequence_variants_from_samples on fetch_Qiita_samples results)
- sample metadata category (EMPO 3 habitat type)
- reference taxonomy used by q2-feature-classifier (e.g., Greengenes or SILVA 138.1)

## Outputs

- class_weights.qza — QIIME 2 artifact of semantic type FeatureWeights, one per EMPO 3 habitat

## How to apply

After q2-clawback has summed classified read counts per reference taxon across all samples sharing an EMPO 3 habitat, normalized them to relative frequencies, and added the uniform 1e-6 unobserved-weight offset, register the resulting per-taxon weight vector as a QIIME 2 artifact with semantic type FeatureWeights and write it to class_weights.qza. Repeat for each of the 14 EMPO 3 habitat types so each habitat yields its own FeatureWeights artifact. Because the artifact format (not raw tables) is what q2-feature-classifier and the readytowear release consume, serialization is the contract point of the workflow: downstream steps must load the .qza via QIIME 2's artifact API rather than parsing raw count files. Validate on load that the artifact declares FeatureWeights and that the payload is a non-negative weight vector.

## Related tools

- **q2-clawback** (plugin whose generate_class_weights action produces the weight vectors that get serialized as FeatureWeights artifacts) — https://github.com/BenKaehler/q2-clawback
- **QIIME 2** (framework providing the artifact (.qza) format and FeatureWeights semantic type used for serialization and later loading) — https://qiime2.org
- **q2-feature-classifier** (downstream consumer that applies the serialized FeatureWeights to train habitat-specific naive Bayes classifiers) — https://github.com/qiime2/q2-feature-classifier/
- **readytowear** (repository of ready-made clawback-generated taxonomic weights; shares the same serialized FeatureWeights artifact contract) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime clawback generate-class-weights --i-classification classification.qza --m-metadata-file sample_metadata.tsv --m-metadata-column empo_3 --o-class-weights class_weights.qza
```

## Evaluation signals

- The output class_weights.qza exists and loads as a QIIME 2 artifact of semantic type FeatureWeights.
- All weights in the artifact are non-negative.
- Taxa observed in the habitat dominate the weight vector, while never-observed taxa sit at the 1e-6 unobserved-weight offset level.
- Each of the 14 EMPO 3 habitat types yields its own serialized FeatureWeights artifact rather than a single pooled vector.

## Limitations

- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation), so serialized weights inherit any upstream reference errors.
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into the weights.
- For natural sample types lacking sufficient characterization, bespoke weights cannot be assembled — average weights estimated from global microbial species distributions are superior to uniform weights in that case.

## Evidence

- [abstract] q2-clawback provides a straightforward alternative for samples from common environments.: "q2-clawback provides a straightforward alternative for samples from common environments."
- [methods] Serialize each habitat's weight vector as a QIIME 2 artifact of semantic type FeatureWeights to class_weights.qza.: [evidence span withheld — anchor status: not_found]
- [readme] This is a QIIME 2 plugin. For details on QIIME 2, see https://qiime2.org.: "This is a QIIME 2 plugin. For details on QIIME 2, see https://qiime2.org."
- [readme] Ready-made taxonomic weights generated by q2-clawback for use with the q2-feature-classifier taxonomy classifier.: [evidence span withheld — anchor status: external_doc]
