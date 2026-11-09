---
name: site-level-recaptured-species-comparison
description: Use when validating curated vs. uncurated OTU tables against paired field-survey
  species lists to compute per-site proportions of recaptured, unregistered, redundant,
  and lost species.
when_to_use_negative:
- The goal is to identify single rare species in community data sets — the article
  notes LULU curation 'may thus not be suitable for identifying single rare species'
- No paired ground-truth species inventory (e.g., field survey list) exists for the
  sampled sites, so recaptured/lost species proportions cannot be defined
- The marker region or organism group lacks reference database coverage comparable
  to GenBank ITS2 for the surveyed flora
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LULU
  role: Post-clustering curation of the OTU tables whose before/after species composition
    is compared at each site
  repo: https://github.com/tobiasgf/lulu
- name: GenBank
  role: Public reference database used to assign OTUs to species and label imperfect
    reference matches
- name: VSEARCH
  role: One of the representative clustering methods producing OTU tables that are
    curated with LULU and compared per site
- name: SWARM
  role: Greedy clustering method producing OTU tables for the before/after curation
    comparison
  repo: https://github.com/torognes/swarm
- name: CROP
  role: Clustering method producing OTU tables for the per-site composition comparison
- name: DADA2
  role: Initial OTU definition method whose tables are curated with LULU and compared
    per site
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:5507cdaa5632f08931351bca4133245cc265c5bac2e41df3a719d87267de5467
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - GenBank
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# site-level-recaptured-species-comparison

## Summary

Quantifies how LULU post-clustering curation changes per-site taxonomic composition by comparing OTU-derived species inventories (via GenBank ITS2 reference assignment) against paired vascular plant field-survey species lists for the 130 Danish sites, for each clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level.

## When to use

Use when validating curated vs. uncurated OTU tables against paired field-survey species lists to compute per-site proportions of recaptured, unregistered, redundant, and lost species.

## When NOT to use

- The goal is to identify single rare species in community data sets — the article notes LULU curation 'may thus not be suitable for identifying single rare species'
- No paired ground-truth species inventory (e.g., field survey list) exists for the sampled sites, so recaptured/lost species proportions cannot be defined
- The marker region or organism group lacks reference database coverage comparable to GenBank ITS2 for the surveyed flora

## Inputs

- LULU-curated OTU tables for VSEARCH, SWARM, CROP, and DADA2 clustering methods at each similarity level (samples as columns, OTUs as rows, unique OTU ids as row names)
- Uncurated OTU tables from the same clustering methods for before-curation comparison
- Per-site vascular plant field survey species lists for the 130 Danish field sites
- GenBank ITS2 reference data for assigning OTUs to species (covers 88% of surveyed species)

## Outputs

- table2_species_validation_metrics.csv with per-site and per-method/level mean proportions of imperfect reference matches, recaptured species, unregistered species, redundant species, and lost recaptured species, before and after LULU curation

## How to apply

Load the LULU-curated OTU tables (default settings: minimum_match = 84, minimum_ratio = 1 (min), minimum_relative_cooccurence = 0.95) for each initial clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level, together with the per-site vascular plant field survey species lists for the 130 Danish sites. Assign OTUs to species using GenBank ITS2 reference data, labeling OTUs whose best matches are imperfect reference matches. For each site and each table, compare the OTU-derived species inventory with the survey site list and count imperfect reference matches, recaptured species (OTUs with a perfect database match uniquely annotated to a plant species recorded in the survey), unregistered species, redundant species, and lost recaptured species. Convert counts to per-site proportions and average them across sites for each method/similarity-level combination. Write the metrics table to table2_species_validation_metrics.csv and verify that curation raises the mean recaptured species proportion from ~0.24 to ~0.44 while lost recaptured species stays ~0.02.

## Related tools

- **LULU** (Post-clustering curation of the OTU tables whose before/after species composition is compared at each site) — https://github.com/tobiasgf/lulu
- **GenBank** (Public reference database used to assign OTUs to species and label imperfect reference matches)
- **VSEARCH** (One of the representative clustering methods producing OTU tables that are curated with LULU and compared per site)
- **SWARM** (Greedy clustering method producing OTU tables for the before/after curation comparison) — https://github.com/torognes/swarm
- **CROP** (Clustering method producing OTU tables for the per-site composition comparison)
- **DADA2** (Initial OTU definition method whose tables are curated with LULU and compared per site)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name)
```

## Evaluation signals

- Mean recaptured species proportion across the 130 sites rises from ~0.24 before curation to ~0.44 after LULU curation
- Mean lost recaptured species proportion remains ~0.02 after curation, matching reported Table 2 results
- Every OTU-derived species assignment is traceable to a GenBank ITS2 match, with perfect matches uniquely annotated to survey species distinguished from imperfect reference matches
- The output table2_species_validation_metrics.csv contains one row per method/similarity-level combination with per-site proportions averaged across all 130 sites
- Per-site proportions are computed from counts relative to the site's field-survey species richness (per-site richness ranged from 6 to 93 in the source study)

## Limitations

- The 'cost of curation' is a small loss of 'real' (rare, low-abundance) species, reflected in the ~0.02 lost recaptured species proportion
- GenBank contained ITS2 data for only 88% of species recorded in the field survey, limiting recapture detection
- LULU curation may not be suitable for identifying single rare species in community data sets
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and β-diversity measures), so its before/after metrics are less informative

## Evidence

- [full_text] recaptured species definition and LULU defaults: [evidence span withheld — anchor status: not_found]
- [intro] GenBank assignment for validation: "For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank)"
- [methods] co-occurrence threshold default: "If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%"
- [readme] LULU input requirements: "The required input of **LULU** is an OTU table and a corresponding matchlist with all the internal matches of OTUs."
- [intro] survey ground truth scope: "we used a plant data set for ITS2 (nuclear ribosomal internal transcribed spacer region 2) obtained from an extensive soil sampling across 130 field sites in Denmark"
- [discussion] cost of curation: "The ‘cost of curation’ is a small loss of ‘real species’"
