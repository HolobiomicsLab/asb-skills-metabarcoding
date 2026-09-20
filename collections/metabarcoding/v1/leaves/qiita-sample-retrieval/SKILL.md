---
name: qiita-sample-retrieval
description: Use when a taxonomy-classification experiment needs environment-specific
  (bespoke) taxonomic weights or empirical test samples, and the source environment
  can be matched to Qiita metadata categories such as EMPO 3 habitats.
when_to_use_negative:
- The sample source environment lacks sufficient Qiita characterization for bespoke
  weight assembly — the paper recommends global average weights instead
- The query sample is from a highly unusual distribution (e.g., synthetic populations)
  where no Qiita habitat match exists — compile custom bespoke weights from existing
  samples instead
- You only need precomputed weights, not samples — use the readytowear repository
  weights rather than re-fetching Qiita data
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: q2-clawback
  role: Provides fetch_Qiita_samples and sequence_variants_from_samples actions that
    perform the Qiita sample retrieval and feature extraction
  repo: https://github.com/BenKaehler/q2-clawback
- name: redbiom
  role: Backend used by fetch_Qiita_samples for rapid sample discovery and feature-characterized
    retrieval from the Qiita cache
  repo: https://github.com/biocore/redbiom
- name: summarize_Qiita_metadata_category_and_contexts
  role: Enumerates the Qiita metadata category and available processing contexts before
    fetching
  repo: https://github.com/BenKaehler/q2-clawback
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:6b16db6adcb0bffa3233915370f3f081cef4a1851f94936f0b2688405a27d33c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - fetch_Qiita_samples
  - imbalanced-learn
  - q2-clawback
  - redbiom
  - scikit-learn
  - summarize_Qiita_metadata_category_and_contexts
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# qiita-sample-retrieval

## Summary

Retrieve environment-matched empirical samples and their sequence variants from Qiita via q2-clawback's fetch_Qiita_samples (redbiom-backed discovery), providing the empirical taxonomic distributions needed to assemble bespoke class weights.

## When to use

Use when a taxonomy-classification experiment needs environment-specific (bespoke) taxonomic weights or empirical test samples, and the source environment can be matched to Qiita metadata categories such as EMPO 3 habitats.

## When NOT to use

- The sample source environment lacks sufficient Qiita characterization for bespoke weight assembly — the paper recommends global average weights instead
- The query sample is from a highly unusual distribution (e.g., synthetic populations) where no Qiita habitat match exists — compile custom bespoke weights from existing samples instead
- You only need precomputed weights, not samples — use the readytowear repository weights rather than re-fetching Qiita data

## Inputs

- Qiita metadata category and value defining the target habitat (e.g., EMPO 3 category, habitat such as soil-non-saline or human-stool)
- redbiom processing context matching the amplicon processing (e.g., Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4)
- Reference sequence database with taxonomy (e.g., Greengenes or SILVA) to pair with fetched features

## Outputs

- Fetched Qiita sample set with sequence variants (feature table / sample data)
- Extracted sequence variants consumable by q2-feature-classifier and generate_class_weights
- Per-sample classified taxonomic abundances (e.g., classified_abundances.csv) when classified before weight assembly

## How to apply

First enumerate the Qiita metadata category and available processing contexts for the target habitat with summarize_Qiita_metadata_category_and_contexts (e.g., the 14 EMPO 3 habitats, yielding 21,513 samples). Then call fetch_Qiita_samples, which uses redbiom to discover and retrieve the matching samples and their sequence variants into a BIOM-style sample set, selecting the context that matches your marker-gene processing (e.g., 16S V4 Greengenes closed-reference). Extract sequence variants from the fetched samples with sequence_variants_from_samples so they pair with the reference sequence database under a shared taxonomy. These empirical sample sets then feed generate_class_weights for per-habitat weights, or serve as held-out test material simulated via sequence_variants_from_samples for cross-validated accuracy scoring.

## Related tools

- **q2-clawback** (Provides fetch_Qiita_samples and sequence_variants_from_samples actions that perform the Qiita sample retrieval and feature extraction) — https://github.com/BenKaehler/q2-clawback
- **redbiom** (Backend used by fetch_Qiita_samples for rapid sample discovery and feature-characterized retrieval from the Qiita cache) — https://github.com/biocore/redbiom
- **summarize_Qiita_metadata_category_and_contexts** (Enumerates the Qiita metadata category and available processing contexts before fetching) — https://github.com/BenKaehler/q2-clawback

## Examples

```
redbiom search metadata "empo_3 == 'animal-distal-gut'" | redbiom fetch samples --context $ctx --output gut_samples.biom
```

## Evaluation signals

- Fetched sample count matches the expected magnitude (e.g., 21,513 samples across the 14 EMPO 3 habitats; ~122 samples sufficed for saline-surface weights)
- Retrieved feature IDs are compatible with the reference database taxonomy (e.g., Greengenes OTU IDs) and every fetched sample has features in the chosen context
- sequence_variants_from_samples yields a non-empty feature table for each habitat that pairs cleanly with the reference sequences
- Downstream generate_class_weights on the fetched samples produces weights for all target habitats without missing-habitat errors
- Cross-validation built on the fetched set contains five distinct shuffled StratifiedKFold IDs, with every reference sequence and sample in exactly one fold per split

## Limitations

- Empirical species distributions introduce error sources such as misannotated sample types and sequencing biases that propagate into assembled taxonomic weights
- Natural sample types lacking sufficient Qiita characterization cannot have bespoke weights assembled from fetched samples
- Systematic biases from common sequencing techniques are not controlled unless multiple technologies are integrated for microbial distribution estimation

## Evidence

- [other] Fetch the environment-matched Qiita samples for the 14 EMPO 3 habitats using q2-clawback's fetch_Qiita_samples (redbiom-backed sample discovery).: [evidence span withheld — anchor status: not_found]
- [discussion] q2-clawback assembles weights from those samples, producing weights for 14 EMPO 3 habitats across 21,513 samples.: [evidence span withheld — anchor status: not_found]
- [methods] Fetch the empirical sample set with fetch_Qiita_samples and pair it with the reference sequence database so both share a common taxonomy.: [evidence span withheld — anchor status: not_found]
- [methods] please check out our ready-made weights at [readytowear](https://github.com/BenKaehler/readytowear): "please check out our ready-made weights at [readytowear](https://github.com/BenKaehler/readytowear)"
- [readme] redbiom allows rapid finding of samples by features or metadata, retrieval of sample data into BIOM, discovery of metadata categories.: "finding samples by arbitrary metadata searches
* summarizing samples over metadata
* retrieval of sample data into BIOM"
