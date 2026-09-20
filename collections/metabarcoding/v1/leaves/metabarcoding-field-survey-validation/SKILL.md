---
name: metabarcoding-field-survey-validation
description: Use when paired field survey species lists exist for metabarcoding sites
  and you must quantify how OTU curation changes per-site taxonomic composition against
  survey ground truth.
when_to_use_negative:
- No paired botanical survey (or other independent ground-truth inventory) exists
  for the sampled sites, so species validation categories cannot be defined
- The goal is to identify individual rare species in community data — the authors
  note LULU curation may not be suitable for this, and the cost of curation is a small
  loss of real, rare, low-abundance species
- The data are not amplicon-derived OTU tables (e.g. already taxonomically assigned
  species lists or abundance matrices without OTU-level structure)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: LULU
  role: R-package performing the post-clustering curation whose effect is validated
    against field survey data; run with default minimum_match = 84, minimum_ratio
    = 1, minimum_relative_cooccurence = 0.95
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the representative clustering methods producing the initial OTU tables,
    and usable to produce the match list for LULU (--usearch_global --self --id .84)
- name: SWARM
  role: Greedy clustering method producing OTU tables subsequently curated with LULU
    and validated against survey data
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: Clustering method producing OTU tables for the validation comparison (noted
    as suboptimal in the discussion)
- name: DADA2
  role: Initial OTU-definition method producing OTU tables subsequently curated with
    LULU; recommended combined with VSEARCH clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:1b1e7940296939fa53cf4d5963d1d11ef4c8d51fcede9ca4ea89785c0ec24d4c
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metabarcoding-field-survey-validation

## Summary

Validates plant ITS2 metabarcoding OTU tables (before and after LULU curation) against paired vascular plant field survey species lists from 130 Danish sites, quantifying per-site taxonomic composition as recaptured, unregistered, redundant, imperfectly matched, and lost species.

## When to use

Use when paired field survey species lists exist for metabarcoding sites and you must quantify how OTU curation changes per-site taxonomic composition against survey ground truth.

## When NOT to use

- No paired botanical survey (or other independent ground-truth inventory) exists for the sampled sites, so species validation categories cannot be defined
- The goal is to identify individual rare species in community data — the authors note LULU curation may not be suitable for this, and the cost of curation is a small loss of real, rare, low-abundance species
- The data are not amplicon-derived OTU tables (e.g. already taxonomically assigned species lists or abundance matrices without OTU-level structure)

## Inputs

- LULU-curated OTU tables (samples as columns, OTUs as rows, unique OTU IDs) for each clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level
- Un-curated OTU tables for the same methods/levels (for 'before curation' metrics)
- Per-site vascular plant field survey species lists for the 130 Danish field sites
- GenBank ITS2 reference data for assigning OTUs to species (covering 88% of surveyed species)
- Match list of internal OTU matches (query OTU id, matching OTU id, similarity %) used for LULU curation

## Outputs

- table2_species_validation_metrics.csv with per-site counts converted to proportions and averaged across sites for each method/level combination
- Per-site species validation category counts: imperfect reference matches, recaptured species, unregistered species, redundant species, lost recaptured species

## How to apply

Load the curated LULU OTU tables (default settings: minimum_match = 84, minimum_ratio = 1, minimum_relative_cooccurence = 0.95) for each initial clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level, together with the per-site vascular plant field survey species lists for the 130 Danish sites. Assign OTUs to species using GenBank ITS2 reference data to label imperfect reference matches. For each site and each table, compare the OTU-derived species inventory with the survey site list, counting imperfect reference matches, recaptured species (perfect database matches uniquely annotated to a surveyed plant species), unregistered species, redundant species, and lost recaptured species. Convert counts to per-site proportions and average across sites for each method/level combination, writing the metrics to table2_species_validation_metrics.csv. The rationale is that the botanical survey (564 species, 6–93 species per site) provides ground truth for judging whether post-clustering curation yields more realistic biodiversity estimates.

## Related tools

- **LULU** (R-package performing the post-clustering curation whose effect is validated against field survey data; run with default minimum_match = 84, minimum_ratio = 1, minimum_relative_cooccurence = 0.95) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the representative clustering methods producing the initial OTU tables, and usable to produce the match list for LULU (--usearch_global --self --id .84))
- **SWARM** (Greedy clustering method producing OTU tables subsequently curated with LULU and validated against survey data) — https://github.com/frederic-mahe/swarm
- **CROP** (Clustering method producing OTU tables for the validation comparison (noted as suboptimal in the discussion))
- **DADA2** (Initial OTU-definition method producing OTU tables subsequently curated with LULU; recommended combined with VSEARCH clustering)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name)
```

## Evaluation signals

- Mean recaptured species proportion rises from ~0.24 before curation to ~0.44 after curation across clustering methods
- Mean lost recaptured species remains ~0.02, confirming the small 'cost of curation'
- Improvement after curation is most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs
- Output table2_species_validation_metrics.csv contains averaged proportions for all five validation categories per method/level combination
- OTU tables conform to the LULU input requirement: samples as columns, OTUs as rows, unique OTU ids as row names

## Limitations

- GenBank contained ITS2 reference data for only 88% of the species recorded in the field survey, so some recaptured species cannot be perfectly annotated
- The cost of curation is a small loss of real (rare, low-abundance) species; LULU curation may not be suitable for identifying single rare species
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and beta-diversity measures), affecting comparisons involving that method
- Validation was performed only for vascular plants; relevance to other organism groups is inferred but not directly tested
