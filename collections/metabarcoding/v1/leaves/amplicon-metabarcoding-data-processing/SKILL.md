---
name: amplicon-metabarcoding-data-processing
description: Use when an OTU table from VSEARCH, SWARM, CROP or DADA2 inflates richness
  relative to ground-truth survey data and you need to remove artefactual OTUs before
  regressing OTU richness against environmental or survey-based richness.
when_to_use_negative:
- The goal is to identify single rare species in community data sets — LULU curation
  may merge or remove genuinely rare OTUs
- The input is already curated, or the study targets composition/turnover only, where
  discarding the tail of the OTU rank-abundance distribution may be adequate
- CROP-produced tables where OTU picking is already suboptimal (low similarity to
  best GenBank match, poor taxonomic redundancy) — curation gains will not fix upstream
  clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: LULU
  role: R package performing co-occurrence based post-clustering curation of the OTU
    table using the OTU table and match list
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: Produces the OTU table and can generate the internal match list (--usearch_global
    --self --id .84)
- name: SWARM
  role: One of the representative initial clustering methods producing OTU tables
    for curation
  repo: https://github.com/frederic-mahe/swarm
- name: DADA2
  role: Recommended initial OTU definition method preceding clustering and LULU curation
- name: CROP
  role: Representative clustering method producing OTU tables for curation comparison
- name: BLASTn
  role: Alternative match-list producer via makeblastdb + blastn -outfmt '6 qseqid
    sseqid pident' -perc_identity 84
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:9e84ff78e661608a6f79c68bea9d58b6a771768c431cd88f3c2fefdb682a5e98
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# amplicon-metabarcoding-data-processing

## Summary

Post-clustering curation of amplicon OTU tables with LULU, which merges erroneous 'daughter' OTUs into more abundant 'parent' OTUs based on co-occurrence patterns and sequence similarity, validated against paired plant field-survey richness data from 130 Danish sites.

## When to use

Use when an OTU table from VSEARCH, SWARM, CROP or DADA2 inflates richness relative to ground-truth survey data and you need to remove artefactual OTUs before regressing OTU richness against environmental or survey-based richness.

## When NOT to use

- The goal is to identify single rare species in community data sets — LULU curation may merge or remove genuinely rare OTUs
- The input is already curated, or the study targets composition/turnover only, where discarding the tail of the OTU rank-abundance distribution may be adequate
- CROP-produced tables where OTU picking is already suboptimal (low similarity to best GenBank match, poor taxonomic redundancy) — curation gains will not fix upstream clustering

## Inputs

- OTU table with samples as columns, OTUs as rows and unique OTU ids as row names (e.g. otutable_test.txt, 2425 OTUs over 130 samples)
- FASTA file of representative OTU sequences with ids matching the OTU table
- Match list: three columns (query OTU id, matching OTU id, pairwise % similarity)
- Paired field-survey richness data (e.g. vascular plant species richness per site) for correspondence validation

## Outputs

- LULU-curated OTU table (curated_result$curated_table) with artefactual daughters merged into parents
- Per-site OTU richness values from curated and singleton-removed tables
- R2 correspondence values (OTU richness vs plant richness) and a comparison_table.csv of improvements per clustering method

## How to apply

First produce the OTU table (samples as columns, OTUs as rows, unique OTU ids as row names) with any clustering method (VSEARCH, SWARM, CROP, DADA2) plus a FASTA of representative OTU sequences. Generate a three-column match list (query OTU id, matching OTU id, % similarity) with BLASTn (-perc_identity 84, -qcov_hsp_perc 80) or VSEARCH (--id .84 --iddef 1 --userfields query+target+id --maxaccepts 0 --query_cov .9 --maxhits 10). Run lulu() in R with minimum_match=84, minimum_relative_cooccurence=0.95 and minimum_ratio_type='min', minimum_ratio=1, so a daughter is merged only when its co-occurrence with a more abundant parent is explained across at least 95% of shared samples and the abundance ratio threshold holds. For benchmarking, also filter singletons (total read count of one) from the uncurated tables and regress per-site OTU richness against per-site plant species richness (R2) for uncurated, singleton-removed, and LULU-curated tables. LULU merges flagged errors bottom-to-top, preserving total read counts, so curated tables should retain read totals while reducing OTU number.

## Related tools

- **LULU** (R package performing co-occurrence based post-clustering curation of the OTU table using the OTU table and match list) — https://github.com/tobiasgf/lulu
- **VSEARCH** (Produces the OTU table and can generate the internal match list (--usearch_global --self --id .84))
- **SWARM** (One of the representative initial clustering methods producing OTU tables for curation) — https://github.com/frederic-mahe/swarm
- **DADA2** (Recommended initial OTU definition method preceding clustering and LULU curation)
- **CROP** (Representative clustering method producing OTU tables for curation comparison)
- **BLASTn** (Alternative match-list producer via makeblastdb + blastn -outfmt '6 qseqid sseqid pident' -perc_identity 84)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name)
```

## Evaluation signals

- Mean R2 improvement of OTU richness vs plant richness over uncurated tables should be ~0.27 for LULU curation vs ~0.03 for singleton removal
- Curated OTU table must preserve total read counts (errors are merged with parents, not deleted)
- Number of OTUs in the curated table must be lower than the input (2425 OTUs over 130 samples in the example table)
- Match list must be a three-column file (query id, target id, % similarity) with matches above the 84% minimum_match threshold
- Improvement should be most pronounced for the greedy methods VSEARCH and SWARM, which initially had the highest number of OTUs

## Limitations

- The 'cost of curation' is a small loss of 'real' rare, low-abundance species
- LULU curation may not be suitable for identifying single rare species in community data sets
- Algorithm validated only for vascular plants (130 Danish field sites); relevance to other organism groups is inferred, not directly tested
- No validation on mock communities with known genetic content has been performed

## Evidence

- [readme] The required input of **LULU** is an OTU table and a corresponding matchlist with all the internal matches of OTUs.: "The required input of **LULU** is an OTU table and a corresponding matchlist with all the internal matches of OTUs."
- [intro] we developed a co-occurrence based post-clustering curation method, LULU. The LULU algorithm excludes artefactual OTUs without discarding rare, but real OTUs: "we developed a co-occurrence based post-clustering curation method, LULU. The LULU algorithm excludes artefactual OTUs without discarding rare, but real OTUs"
- [discussion] The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs: "The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs"
- [methods] If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%, meaning that 1 in 20: "If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%, meaning that 1 in 20"
- [discussion] After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top: "After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top"
