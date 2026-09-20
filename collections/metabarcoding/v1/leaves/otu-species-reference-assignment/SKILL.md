---
name: otu-species-reference-assignment
description: Reference for validating OTU tables against ground-truth species inventories
  when OTUs must be labeled taxonomically (GenBank ITS2) and compared per-site to
  field survey lists to quantify recaptured, unregistered, redundant, and lost species.
when_to_use_negative:
- The target organism group or locus has poor GenBank ITS2 coverage — assignment quality
  will be dominated by missing references rather than curation effects
- There is no independent ground-truth species inventory (e.g. field survey list)
  to compare OTU-derived inventories against
- Goal is single rare species identification — the article notes LULU curation 'may
  thus not be suitable for identifying single rare species'
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3058
tools:
- name: LULU
  role: Produces the curated OTU tables (default minimum_match=84, minimum_ratio=1,
    minimum_relative_cooccurence=0.95) whose species assignments are compared before
    and after curation
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the initial clustering methods (with SWARM, CROP, DADA2) generating
    the OTU tables; also usable to produce the internal match list for LULU
- name: SWARM
  role: Initial clustering method producing OTU tables for species assignment and
    validation
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: Initial clustering method producing OTU tables for species assignment and
    validation
- name: DADA2
  role: Initial clustering method producing OTU tables for species assignment and
    validation; recommended for initial OTU definition before VSEARCH clustering and
    LULU curation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:ffc06a9573796f0c838d8a19696f7d5ad81145edd1b6a32a4ccd95bd710655bf
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3058
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# otu-species-reference-assignment

## Summary

Assigns OTUs from amplicon tables to species using public reference databases (GenBank ITS2) and classifies each assignment as an imperfect reference match, recaptured species, unregistered species, redundant species, or lost recaptured species relative to field-survey species lists.

## When to use

Reference for validating OTU tables against ground-truth species inventories when OTUs must be labeled taxonomically (GenBank ITS2) and compared per-site to field survey lists to quantify recaptured, unregistered, redundant, and lost species.

## When NOT to use

- The target organism group or locus has poor GenBank ITS2 coverage — assignment quality will be dominated by missing references rather than curation effects
- There is no independent ground-truth species inventory (e.g. field survey list) to compare OTU-derived inventories against
- Goal is single rare species identification — the article notes LULU curation 'may thus not be suitable for identifying single rare species'

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names), curated or uncurated
- Representative OTU sequences (FASTA, e.g. centroids_test.txt with 2425 OTUs over 130 samples)
- GenBank ITS2 reference database for species assignment
- Per-site vascular plant field survey species lists for the 130 Danish sites

## Outputs

- Per-site species classification counts: imperfect reference matches, recaptured species, unregistered species, redundant species, lost recaptured species
- Per-site proportions averaged across sites per clustering method/similarity level
- Metrics table (table2_species_validation_metrics.csv)

## How to apply

Load the OTU tables (curated with LULU at minimum_match=84, minimum_ratio=1, minimum_relative_cooccurence=0.95, or uncurated) and the per-site vascular plant field survey species lists for the 130 Danish sites. Assign each OTU to a species by matching its representative sequence against GenBank ITS2 reference data, flagging perfect database matches, uniquely annotated matches, and imperfect (lower-similarity) reference matches. For each site and table, compare the OTU-derived species inventory with the survey list and count imperfect reference matches, recaptured species (perfect match uniquely annotated to a surveyed species), unregistered species, redundant species, and lost recaptured species. Convert counts to per-site proportions and average across sites for each clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level. Rationale: GenBank contained ITS2 data for 88% of the species recorded in the field survey, so reference assignment quality directly determines how well molecular inventories can be validated against botanical ground truth.

## Related tools

- **LULU** (Produces the curated OTU tables (default minimum_match=84, minimum_ratio=1, minimum_relative_cooccurence=0.95) whose species assignments are compared before and after curation) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the initial clustering methods (with SWARM, CROP, DADA2) generating the OTU tables; also usable to produce the internal match list for LULU)
- **SWARM** (Initial clustering method producing OTU tables for species assignment and validation) — https://github.com/frederic-mahe/swarm
- **CROP** (Initial clustering method producing OTU tables for species assignment and validation)
- **DADA2** (Initial clustering method producing OTU tables for species assignment and validation; recommended for initial OTU definition before VSEARCH clustering and LULU curation)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); curated_table <- curated_result$curated_table  # then BLASTn: blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta
```

## Evaluation signals

- Mean recaptured species proportion rises from ~0.24 before LULU curation to ~0.44 after curation across all clustering methods
- Mean lost recaptured species proportion remains ~0.02 after curation
- GenBank ITS2 reference coverage spans 88% of the species recorded in the 130-site field survey (564 species total, 6–93 species per site)
- Per-site proportions of unregistered and redundant species decrease after curation, consistent with reported Table 2 results
- Output table table2_species_validation_metrics.csv contains one row per method/similarity-level combination with all five species-validation metrics

## Limitations

- Assignment depends on incomplete reference databases; 'imperfect reference matches' arise when the best GenBank hit is not a perfect, uniquely annotated species match
- The 'cost of curation' is a small loss of real (rare, low-abundance) species, reflected in lost recaptured species (~0.02)
- Validation was performed only for vascular plants (ITS2); relevance to other organism groups is inferred but not directly tested
- CROP OTU picking appeared suboptimal, with low similarity to best GenBank match and poor redundancy measures

## Evidence

- [intro] genbank_coverage: "we assigned OTUs to species in public reference databases (GenBank), which contained ITS2 data for 88% of species recorded in the field survey"
- [results] recapture_improvement: [evidence span withheld — anchor status: not_found]
- [discussion] genbank_discussion: "the similarity between OTU and best GenBank match (Fig. 1e) were far lower, and the taxonomic redundancy (Fig. 1c)"
- [intro] survey_scope: "A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93"
- [discussion] rare_species_caveat: "LULU curation may thus not be suitable for identifying single rare species in community data sets"
- [readme] otu_table_format: "The requirements of LULU are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names."
