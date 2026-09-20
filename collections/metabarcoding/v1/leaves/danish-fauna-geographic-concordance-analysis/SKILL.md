---
name: danish-fauna-geographic-concordance-analysis
description: Use when a resolved species list from eDNA metabarcoding (e.g., the 93
  Vestamager-resolved arthropod taxa) needs independent validation against public
  occurrence records at defined geographic levels before ecological interpretation.
when_to_use_negative:
- The taxon list contains only unassigned or higher-level (family/order) OTUs rather
  than species-level assignments, which cannot be matched against species-level faunistic
  records.
- The study area lies outside Danish/database coverage such that Naturbasen and Fauna
  Europaea cannot provide meaningful presence/absence evidence for the sampled locality.
- The goal is to test primer or sequencing performance rather than ecological plausibility
  of an already-resolved species list.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Naturbasen / allearter
  role: Danish national fauna database queried for occurrence records of each eDNA
    taxon at Vestamager and Amager
- name: Fauna Europaea
  role: European faunal distribution database used as a second reference for per-taxon
    occurrence flagging
- name: tobiasgf/Bioinformatic-tools
  role: Bioinformatic tool collection associated with the eDNA metabarcoding workflow
    that produces the upstream species list
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:cf4df8846321a5b05a1e703fd210c1f5d9f6f2ddbf682867692d7a84f37df29e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - Fauna Europaea
  - Naturbasen / allearter
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# danish-fauna-geographic-concordance-analysis

## Summary

Validates eDNA-resolved arthropod taxa against independent Danish faunistic occurrence databases (Naturbasen / allearter, Fauna Europaea) by flagging each species as present or absent at a specific locality (Vestamager) and region (Amager), then tallying concordance percentages as an ecological plausibility check on metabarcoding results.

## When to use

Use when a resolved species list from eDNA metabarcoding (e.g., the 93 Vestamager-resolved arthropod taxa) needs independent validation against public occurrence records at defined geographic levels before ecological interpretation.

## When NOT to use

- The taxon list contains only unassigned or higher-level (family/order) OTUs rather than species-level assignments, which cannot be matched against species-level faunistic records.
- The study area lies outside Danish/database coverage such that Naturbasen and Fauna Europaea cannot provide meaningful presence/absence evidence for the sampled locality.
- The goal is to test primer or sequencing performance rather than ecological plausibility of an already-resolved species list.

## Inputs

- Final resolved eDNA species list (e.g., 93 Vestamager arthropod taxa from flower metabarcoding)
- Naturbasen / allearter occurrence records (Danish national fauna database)
- Fauna Europaea distribution records

## Outputs

- vestamager_occurrence_concordance.csv with per-taxon present/absent flags at Vestamager and Amager
- Summary concordance counts and percentages (e.g., 55/93 = 59% at Vestamager; 68/93 = 73% on Amager)

## How to apply

Load the final eDNA species list (in the source study, the 93 arthropod taxa resolved from wild-flower eDNA at Vestamager, recovered with 16S rRNA and COI primer sets). Query each taxon name against Naturbasen / allearter (Danish national fauna database) and Fauna Europaea, and flag it as present or absent at two nested geographic levels: the exact sampling locality (Vestamager) and the enclosing island (Amager). Tally counts and percentages at each level, then compare against expected concordance ranges — in the source study, 55/93 (59%) taxa at Vestamager and 68/93 (73%) on Amager, with the island-level figure expected to be higher because it is a coarser filter. Write per-taxon flags plus summary counts to vestamager_occurrence_concordance.csv; divergence from the reported statistics signals a taxon-name mapping or database-coverage error that must be reviewed before drawing ecological conclusions.

## Related tools

- **Naturbasen / allearter** (Danish national fauna database queried for occurrence records of each eDNA taxon at Vestamager and Amager)
- **Fauna Europaea** (European faunal distribution database used as a second reference for per-taxon occurrence flagging)
- **tobiasgf/Bioinformatic-tools** (Bioinformatic tool collection associated with the eDNA metabarcoding workflow that produces the upstream species list) — tobiasgf/Bioinformatic-tools

## Examples

```
python concordance_tally.py --taxa vestamager_taxa.csv --databases Naturbasen,FaunaEuropaea --levels Vestamager,Amager --out vestamager_occurrence_concordance.csv
```

## Evaluation signals

- Reproducing the reported concordance statistics of 55/93 (59%) taxa present at Vestamager and 68/93 (73%) on Amager from the per-taxon flags.
- Amager-level concordance percentage must be greater than or equal to the Vestamager-level percentage, since Amager is a coarser geographic filter containing the locality.
- All 93 resolved taxa appear exactly once in vestamager_occurrence_concordance.csv with a present/absent flag at both geographic levels (no unqueried taxa).
- Taxa flagged present at Vestamager must also be flagged present on Amager (nested-consistency check); violations indicate a mapping or database-query error.
- Unmatched taxon names are logged for review rather than silently dropped, since database-coverage gaps can mimic absence.

## Limitations

- Concordance is bounded by database coverage: absence in Naturbasen or Fauna Europaea may reflect incomplete recording rather than true absence, so percentages underestimate true overlap.
- The upstream metabarcoding experiment was not exhaustive — 'an even higher arthropod richness could be obtained using this eDNA approach' — so the concordance set is a subset of the local fauna.
- Divergence from the reported 59%/73% values 'indicates a mapping or database-coverage error requiring review', so counts are only interpretable after taxon-name matching is confirmed.

## Evidence

- [other] Of the 93 Vestamager-resolved eDNA taxa, 55 (59%) have been recorded at the same sampling locality (Vestamager) and 68 (73%) on the island of Amager, indicating good concordance with faunistic records.: [evidence span withheld — anchor status: not_found]
- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators"
- [other] Query occurrence records for each taxon against Naturbasen / allearter (Danish national fauna database) and Fauna Europaea.: [evidence span withheld — anchor status: not_found]
