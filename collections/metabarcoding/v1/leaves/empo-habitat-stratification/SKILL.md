---
name: empo-habitat-stratification
description: Use when a sample set carries an environment/habitat metadata category
  (e.g., EMPO 3) and bespoke, habitat-specific taxonomic weights must be generated
  per environment rather than globally.
when_to_use_negative:
- When no environment/habitat metadata category is available for the samples, or the
  sample type lacks sufficient characterization for bespoke weight assembly — use
  average weights estimated from global microbial species distributions instead of
  uniform weights
- For highly unusual sample distributions (e.g., synthetic populations) where compiling
  custom bespoke weights from existing samples is required rather than habitat-level
  stratification
- When ready-made weights already exist for the primer/database/habitat combination
  — use the readytowear inventory instead of re-running clawback
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: q2-clawback
  role: Plugin whose generate_class_weights action aggregates, normalizes, and offsets
    classified read counts within each EMPO 3 habitat stratum
  repo: https://github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework hosting the plugin and the FeatureWeights artifact semantic type
    used to store per-habitat weights
- name: q2-feature-classifier
  role: Downstream consumer of the habitat-specific weights for naive Bayes taxonomic
    classification
- name: redbiom
  role: Backend used by fetch_Qiita_samples to discover and retrieve samples for a
    metadata category such as EMPO 3
  repo: https://github.com/biocore/redbiom
- name: readytowear
  role: Repository of pre-generated per-EMPO 3 habitat weights produced by this skill,
    usable as a shortcut
  repo: https://github.com/BenKaehler/readytowear
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:dc52064ec345d70a682b6758ac5c0d8b80e501f204afc940da332e639845dc34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  - redbiom
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# empo-habitat-stratification

## Summary

Stratifies fetched Qiita samples by their EMPO 3 habitat metadata category so that per-habitat taxonomic weights can be assembled with q2-clawback, yielding habitat-specific (e.g., 14 EMPO 3 habitat type) weights for q2-feature-classifier.

## When to use

Use when a sample set carries an environment/habitat metadata category (e.g., EMPO 3) and bespoke, habitat-specific taxonomic weights must be generated per environment rather than globally.

## When NOT to use

- When no environment/habitat metadata category is available for the samples, or the sample type lacks sufficient characterization for bespoke weight assembly — use average weights estimated from global microbial species distributions instead of uniform weights
- For highly unusual sample distributions (e.g., synthetic populations) where compiling custom bespoke weights from existing samples is required rather than habitat-level stratification
- When ready-made weights already exist for the primer/database/habitat combination — use the readytowear inventory instead of re-running clawback

## Inputs

- Per-sample classified sequence-variant counts (output of sequence_variants_from_samples applied to fetch_Qiita_samples results)
- Habitat metadata category (e.g., EMPO 3) for the fetched samples
- Curated reference taxonomy (e.g., Greengenes or SILVA 138.1)

## Outputs

- Per-habitat weight vectors serialized as QIIME 2 artifacts of semantic type FeatureWeights (class_weights.qza)
- Habitat-stratified weight collection covering the 14 EMPO 3 habitat types (or a selected subset such as animal-distal-gut, soil-non-saline, human-oral, human-stool)

## How to apply

Load the per-sample classified sequence-variant counts (produced by sequence_variants_from_samples on fetch_Qiita_samples output) together with the EMPO 3 metadata category. Partition samples into groups by their EMPO 3 habitat value, then for each habitat call q2-clawback's generate_class_weights to sum classified read counts per reference taxon across all samples in that habitat. Normalize the summed per-taxon counts to relative frequencies and add a uniform offset (default unobserved weight 1e-6) so every reference species retains nonzero weight. Serialize each habitat's weight vector as a QIIME 2 FeatureWeights artifact (class_weights.qza). Rationale: the more specific the taxonomic weights are to a query sample's environment, the better the classification accuracy, and even deliberately incorrect cross-habitat weights outperformed uniform weights.

## Related tools

- **q2-clawback** (Plugin whose generate_class_weights action aggregates, normalizes, and offsets classified read counts within each EMPO 3 habitat stratum) — https://github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework hosting the plugin and the FeatureWeights artifact semantic type used to store per-habitat weights)
- **q2-feature-classifier** (Downstream consumer of the habitat-specific weights for naive Bayes taxonomic classification)
- **redbiom** (Backend used by fetch_Qiita_samples to discover and retrieve samples for a metadata category such as EMPO 3) — https://github.com/biocore/redbiom
- **readytowear** (Repository of pre-generated per-EMPO 3 habitat weights produced by this skill, usable as a shortcut) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime clawback generate-class-weights --i-classification fetched_classifications.qza --m-metadata-file empo3.tsv --m-metadata-column empo3 --o-classification-weights class_weights.qza
```

## Evaluation signals

- class_weights.qza exists for each EMPO 3 habitat and loads as semantic type FeatureWeights
- All weight values are non-negative; observed taxa dominate while unobserved taxa sit at the 1e-6 unobserved-weight offset level
- Per-habitat weights sum to 1 after normalization to relative frequencies
- Cross-validation against the target habitat shows improved species-level accuracy versus uniform weights (species-level average error rates decline from 25% to 14%)
- Every reference species retains nonzero weight in each habitat's vector

## Limitations

- Performance degradation was observed for rare species under bespoke weighting
- Empirical species distributions introduce error sources such as misannotated sample types and sequencing biases propagated into the taxonomic weights
- Natural sample types lacking sufficient characterization cannot have bespoke weights assembled; systematic sequencing-technology bias is not controlled without integrating multiple technologies

## Evidence

- [methods] habitat-specific weights via clawback: [evidence span withheld — anchor status: not_found]
- [discussion] specificity improves accuracy: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [discussion] cross-habitat beats uniform: "these weights still outperformed uniform weights for species classification in the majority of cases"
- [readme] readytowear shortcut: [evidence span withheld — anchor status: external_doc]
- [abstract] species-level gains: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
