---
name: curated-otu-table-quality-metrics
description: Use when you have metabarcoding OTU tables (e.g. ITS2) paired with an
  independent species inventory (field survey lists) and need to measure whether curation
  (LULU, singleton removal, dbotu3) changes species-level composition.
when_to_use_negative:
- There is no paired ground-truth species inventory (e.g. field survey list) to validate
  the OTU-derived inventories against
- The goal is alpha/beta diversity or taxonomic composition per se rather than assessing
  whether curation changed species-level fidelity relative to a survey
- The OTUs cannot be assigned to species because reference databases lack ITS2 (or
  equivalent marker) data for the taxa
edam_operation: http://edamontology.org/operation_3478
tools:
- name: LULU
  role: Post-clustering curation of the OTU tables whose before/after species-composition
    metrics are compared
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: Initial clustering method producing OTU tables to validate, and optionally
    used (--usearch_global --self) to build the match list
- name: SWARM
  role: Initial clustering method producing OTU tables to validate
- name: CROP
  role: Initial clustering method producing OTU tables to validate
- name: DADA2
  role: Initial OTU definition method producing OTU tables to validate
- name: BLASTn
  role: Produces the OTU match list (makeblastdb + blastn with -perc_identity 84)
    consumed by LULU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:1604be04476ee88f36f57ba81a0c7a8c49ae7d28ab657d5bdc965db913177ae5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - CROP
  - DADA2
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3478
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# curated-otu-table-quality-metrics

## Summary

Quantifies how LULU post-clustering curation changes per-site taxonomic composition of OTU tables by classifying OTU-derived species against paired vascular plant field survey lists as recaptured, imperfect reference matches, unregistered, redundant, or lost recaptured species. Provides the validation logic (Table 2) for judging whether curation improves species-level fidelity.

## When to use

Use when you have metabarcoding OTU tables (e.g. ITS2) paired with an independent species inventory (field survey lists) and need to measure whether curation (LULU, singleton removal, dbotu3) changes species-level composition.

## When NOT to use

- There is no paired ground-truth species inventory (e.g. field survey list) to validate the OTU-derived inventories against
- The goal is alpha/beta diversity or taxonomic composition per se rather than assessing whether curation changed species-level fidelity relative to a survey
- The OTUs cannot be assigned to species because reference databases lack ITS2 (or equivalent marker) data for the taxa

## Inputs

- LULU-curated and uncurated OTU tables (OTUs as rows, samples as columns, unique OTU ids as row names) for VSEARCH, SWARM, CROP, and DADA2 at each similarity level
- Match list of internal OTU matches (query OTU id, target OTU id, percent similarity) produced with BLASTn or VSEARCH
- GenBank ITS2 reference assignments labeling each OTU with its best species match
- Per-site vascular plant field survey species lists for the 130 Danish sites (564 species total, 6-93 species per site)

## Outputs

- table2_species_validation_metrics.csv: per-site and method/level-averaged proportions of imperfect reference matches, recaptured species, unregistered species, redundant species, and lost recaptured species, before and after LULU curation

## How to apply

Load the curated LULU OTU tables (default settings: minimum_match = 84, minimum_ratio = 1 (min), minimum_relative_cooccurence = 0.95) for each initial clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level, plus the per-site vascular plant survey lists for the 130 Danish sites. Assign OTUs to species using GenBank ITS2 reference data, labeling OTUs whose best database match is not exactly a survey species as 'imperfect reference matches'. For each site and table, compare the OTU-derived species inventory with the survey site list, counting imperfect reference matches, recaptured species (perfect match uniquely annotated to a surveyed species), unregistered species, redundant species, and lost recaptured species. Convert counts to per-site proportions, average across sites for each method/level combination, and write table2_species_validation_metrics.csv. Rationale: curation merges co-occurring, sequence-similar 'daughter' OTUs into more abundant 'parents', so validating against ground truth confirms real species are retained while errors are merged.

## Related tools

- **LULU** (Post-clustering curation of the OTU tables whose before/after species-composition metrics are compared) — https://github.com/tobiasgf/lulu
- **VSEARCH** (Initial clustering method producing OTU tables to validate, and optionally used (--usearch_global --self) to build the match list)
- **SWARM** (Initial clustering method producing OTU tables to validate)
- **CROP** (Initial clustering method producing OTU tables to validate)
- **DADA2** (Initial OTU definition method producing OTU tables to validate)
- **BLASTn** (Produces the OTU match list (makeblastdb + blastn with -perc_identity 84) consumed by LULU)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); curated_table <- curated_result$curated_table
```

## Evaluation signals

- Mean recaptured species proportion rises from ~0.24 before curation to ~0.44 after curation across clustering methods
- Mean lost recaptured species remains ~0.02 after curation (small cost of curation)
- Output metrics table contains per-site proportions in [0,1] averaged across the 130 sites for every method/similarity-level combination
- OTU tables conform to LULU's expected format (samples as columns, OTUs as rows with unique OTU ids) and the match list has three columns: query id, target id, percent similarity
- Uncurated vs curated comparisons show reduced taxonomic redundancy (fewer OTUs per species) without losing surveyed species

## Limitations

- The cost of curation is a small loss of real (rare, low-abundance) species, so curation may not be suitable for identifying single rare species
- Validation relied on GenBank ITS2 data covering 88% of surveyed species, so imperfect reference matches partly reflect incomplete references rather than errors
- The algorithm was validated only on vascular plants; relevance to other organism groups is inferred but not directly tested
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy), affecting its validation metrics

## Evidence

- [intro] To validate the LULU algorithm, we used a plant data set for ITS2 obtained from an extensive soil sampling across 130 field sites in Denmark: "To validate the LULU algorithm, we used a plant data set for ITS2 (nuclear ribosomal internal transcribed spacer region 2) obtained from an extensive soil sampling across 130 field sites in Denmark,"
- [intro] we assigned OTUs to species in public reference databases (GenBank), which contained ITS2 data for 88% of species recorded in the field survey: "we assigned OTUs to species in public reference databases (GenBank), which contained ITS2 data for 88% of species recorded in the field survey"
- [abstract] Here we present and validate a method, called LULU, for removing erroneous molecular operational taxonomic units (OTUs) from community data: "Here we present and validate a method, called LULU, for removing erroneous molecular operational taxonomic units (OTUs) from community data"
- [discussion] The cost of curation is a small loss of real species: "The ‘cost of curation’ is a small loss of ‘real species’"
- [discussion] LULU curation may thus not be suitable for identifying single rare species in community data sets: "LULU curation may thus not be suitable for identifying single rare species in community data sets"
- [readme] The required input of LULU is an OTU table and a corresponding matchlist with all the internal matches of OTUs.: "The required input of **LULU** is an OTU table and a corresponding matchlist with all the internal matches of OTUs."
