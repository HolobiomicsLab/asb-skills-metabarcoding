---
name: dbotu3-abundance-threshold-clustering
description: Use when you need an alternative to LULU for post-clustering curation
  of OTU tables (e.g., from VSEARCH, SWARM, CROP, DADA2), or to benchmark curation
  methods against biodiversity ground truth.
when_to_use_negative:
- When the OTU table lacks representative centroid sequences — dbotu3's genetic dissimilarity
  criterion (0.16) requires pairwise sequence comparison among OTUs
- When the goal is to identify individual rare species rather than community-level
  biodiversity estimates — post-clustering curation carries a small loss of real rare
  species
- When the analysis goal is one-step clustering of raw reads with no benchmark against
  LULU or ground truth — the validated evidence here is specifically for curation
  benchmarking on amplicon OTU tables
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: dbotu3
  role: Core algorithm run both for one-step clustering as intended and as an alternative
    to LULU for post-clustering curation, with genetic dissimilarity 0.16 and abundance
    criteria 0 and 10
- name: LULU
  role: Reference post-clustering curation method against which dbotu3 curation results
    are benchmarked on the same curation metrics
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: Produces initial OTU tables (and can produce the match list) that serve as
    input for curation benchmarking
- name: SWARM
  role: Produces initial OTU tables for the curation benchmarking comparison
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: Produces initial OTU tables for the curation benchmarking comparison
- name: DADA2
  role: Produces initial OTU tables for the curation benchmarking comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:8605c98de6f381bb82c13d0d09677d520f024bd1e20e584555f552b7bd409953
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - LULU
  - SWARM
  - VSEARCH
  - dbotu3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# dbotu3-abundance-threshold-clustering

## Summary

Applies the dbotu3 algorithm with an abundance-threshold criterion both for one-step OTU clustering as intended and as a post-clustering curation step, benchmarked against LULU curation on plant ITS2 OTU tables validated against field survey data.

## When to use

Use when you need an alternative to LULU for post-clustering curation of OTU tables (e.g., from VSEARCH, SWARM, CROP, DADA2), or to benchmark curation methods against biodiversity ground truth.

## When NOT to use

- When the OTU table lacks representative centroid sequences — dbotu3's genetic dissimilarity criterion (0.16) requires pairwise sequence comparison among OTUs
- When the goal is to identify individual rare species rather than community-level biodiversity estimates — post-clustering curation carries a small loss of real rare species
- When the analysis goal is one-step clustering of raw reads with no benchmark against LULU or ground truth — the validated evidence here is specifically for curation benchmarking on amplicon OTU tables

## Inputs

- Initial OTU tables (samples as columns, OTUs as rows with unique OTU ids) produced by VSEARCH, SWARM, CROP, or DADA2
- FASTA files of representative (centroid) sequences for each OTU
- Field survey species data used as ground truth for validation
- GenBank ITS2 reference data for assigning OTUs to species (best-match validation)

## Outputs

- Curated OTU tables produced by dbotu3 in one-step clustering mode and in post-clustering curation mode (for abundance criteria 0 and 10)
- Benchmark table of curation validation metrics (OTU counts, alpha-diversity, taxonomic redundancy, average best-match similarity) comparing dbotu3 with LULU

## How to apply

Load the initial OTU tables produced from the plant ITS2 soil dataset (130 Danish sites) by representative clustering methods (VSEARCH, SWARM, CROP, DADA2), together with each table's representative OTU sequences. Run dbotu3 on each table with genetic dissimilarity set to 0.16 and abundance criteria of 0 and 10, in two modes: the intended 'one-step' clustering mode, and as post-clustering curation applied to pre-existing OTU tables. For every curated table, compute the same curation validation metrics used for LULU — OTU counts, alpha-diversity, taxonomic redundancy, and similarity to the best GenBank match — against the field survey ground truth (564 species recorded across the 130 sites). Assemble the metrics into a benchmark comparison mirroring the supplementary figures, contrasting dbotu3 results with the corresponding LULU curation results.

## Related tools

- **dbotu3** (Core algorithm run both for one-step clustering as intended and as an alternative to LULU for post-clustering curation, with genetic dissimilarity 0.16 and abundance criteria 0 and 10)
- **LULU** (Reference post-clustering curation method against which dbotu3 curation results are benchmarked on the same curation metrics) — https://github.com/tobiasgf/lulu
- **VSEARCH** (Produces initial OTU tables (and can produce the match list) that serve as input for curation benchmarking)
- **SWARM** (Produces initial OTU tables for the curation benchmarking comparison) — https://github.com/frederic-mahe/swarm
- **CROP** (Produces initial OTU tables for the curation benchmarking comparison)
- **DADA2** (Produces initial OTU tables for the curation benchmarking comparison)

## Evaluation signals

- For both abundance settings (0 and 10), dbotu3 applied as post-clustering curation should improve the validation metrics relative to the uncurated input table
- In the benchmark, LULU should perform better than dbotu3 in all comparisons except a few metrics — notably the average best match for the CROP tables — a divergence from this pattern signals a pipeline or parameter error
- Curated tables should show reduced OTU counts and improved taxonomic redundancy (fewer species represented by multiple OTUs) relative to uncurated tables
- Alpha-diversity estimates from curated tables should be closer to the field survey ground truth (per-site richness 6–93 species, 564 species total) than uncurated estimates
- Total read count should be preserved through curation, since erroneous OTUs are merged with their parent OTUs rather than discarded

## Limitations

- LULU outperformed dbotu3 in nearly all benchmark comparisons, so dbotu3 curation is not generally the stronger option
- Post-clustering curation carries a small 'cost of curation': loss of some real (rare, low-abundance) species, and may not be suitable for identifying single rare species in community datasets
- The validation was performed only on a plant ITS2 soil dataset from 130 Danish field sites; relevance to other organism groups is inferred but not directly tested

## Evidence

- [intro] We also tested the dbotu3 algorithm, both for 'one-step' clustering as intended, and as an alternative to LULU for post-clustering curation: "We also tested the dbotu3 algorithm, both for ‘one-step’ clustering as intended, and as an alternative to LULU for post-clustering curation"
- [intro] OTU tables were produced using representative clustering methods: "OTU tables were produced from the sequence data using a set of representative clustering methods: VSEARCH22, SWARM23, CROP24 and DADA213"
- [readme] LULU README: package purpose: "A r-package for distribution based post clustering curation of amplicon data."
- [readme] LULU README: co-occurrence based error identification: "By evaluating the co-occurence patterns of OTUs among samples **LULU** identifies OTUs that consistently satisfy some user selected criteria for being errors"
