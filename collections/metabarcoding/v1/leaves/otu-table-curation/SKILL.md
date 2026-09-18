---
name: otu-table-curation
description: Use when an OTU/ASV table from any clustering method (VSEARCH, SWARM,
  CROP, DADA2) shows inflated richness or taxonomic redundancy (e.g., many OTUs mapping
  to single-species genera like Fagus) and needs curation before α-diversity, redundancy,
  or beta-diversity analyses.
when_to_use_negative:
- 'The goal is to detect or retain single rare species: the article states LULU curation
  may not be suitable for identifying single rare species, and the ''cost of curation''
  is a small loss of real rare, low-abundance species.'
- No representative OTU sequences are available to build a match list, or samples/OTUs
  are too few for meaningful co-occurrence patterns across a multi-sample data set.
- The table is already curated (e.g., a previously LULU-curated table), where a second
  pass would further collapse real diversity without new information.
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: LULU
  role: 'R function performing the co-occurrence-based post-clustering curation: flagging
    daughter OTUs and merging them into parents from the OTU table plus match list'
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Generates the match list of internal OTU sequence similarities prior to curation
    (makeblastdb -parse_seqids -dbtype nucl; blastn -outfmt '6 qseqid sseqid pident'
    -qcov_hsp_perc 80 -perc_identity 84)
- name: VSEARCH
  role: Alternative pairwise matcher for producing the match list (--usearch_global
    --self --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10); also used
    for initial OTU clustering
merged_aliases:
- amplicon-otu-table-curation
- otu-table-curation-validation
- otu-table-curation-read-count-extraction
merged_alias_records:
- alias: amplicon-otu-table-curation
  slug: amplicon-otu-table-curation
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
- alias: otu-table-curation-validation
  slug: otu-table-curation-validation
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
- alias: otu-table-curation-read-count-extraction
  slug: otu-table-curation-read-count-extraction
  cosine_score: 0.8537
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:754b2cfbbbec6f2ce2e3867cd7d3672534ba871145fd49003c2cc3fd1a723bf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - CROP
  - DADA2
  - LULU
  - R
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# otu-table-curation

## Summary

Post-clustering curation of amplicon OTU tables using the LULU algorithm, which identifies erroneous 'daughter' OTUs via sequence similarity plus co-occurrence patterns and merges them into more abundant 'parent' OTUs to yield more realistic biodiversity metrics.

## When to use

Use when an OTU/ASV table from any clustering method (VSEARCH, SWARM, CROP, DADA2) shows inflated richness or taxonomic redundancy (e.g., many OTUs mapping to single-species genera like Fagus) and needs curation before α-diversity, redundancy, or beta-diversity analyses.

## When NOT to use

- The goal is to detect or retain single rare species: the article states LULU curation may not be suitable for identifying single rare species, and the 'cost of curation' is a small loss of real rare, low-abundance species.
- No representative OTU sequences are available to build a match list, or samples/OTUs are too few for meaningful co-occurrence patterns across a multi-sample data set.
- The table is already curated (e.g., a previously LULU-curated table), where a second pass would further collapse real diversity without new information.

## Inputs

- OTU table with samples as columns, unique OTU ids as row names (e.g., otutable_test.txt, 2425 OTUs over 130 samples)
- FASTA file with representative/centroid sequence of each OTU (e.g., centroids_test.txt)
- Match list: 3-column file of query OTU id, matching OTU id, and pairwise similarity score (%) from BLASTn or VSEARCH
- R (>=3.3.2 development context) with the lulu package installed via devtools::install_github('tobiasgf/lulu')

## Outputs

- Curated OTU table (curated_result$curated_table, e.g., curated_table.txt)
- Curation summary object with curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, and an otu_map (columns: total, spread, parent_id, curated, rank)

## How to apply

Produce a 3-column match list (query id, target id, percent identity) from the OTU centroid sequences with BLASTn (-outfmt '6 qseqid sseqid pident', -qcov_hsp_perc 80, -perc_identity 84) or VSEARCH --usearch_global (--id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10). In R, supply the OTU table (samples as columns, unique OTU ids as row names) and match list to the lulu() function. The algorithm sorts OTUs by decreasing occurrence (secondarily by total read count), treats each OTU top-to-bottom as a potential daughter, selects hits within the dissimilarity threshold, keeps only potential parents with occurrence equal to or higher than the daughter, and flags the daughter as an error of the first parent satisfying minimum_relative_cooccurence (default 0.95) and minimum_ratio (default 1, evaluated as 'min' or 'avg'). Flagged errors are merged with their designated parents bottom-to-top so total read counts are preserved. Defaults validated on the tutorial data (2425 OTUs over 130 samples) are minimum_match 84, minimum_relative_cooccurence 0.95, minimum_ratio 1 with minimum_ratio_type 'min'.

## Related tools

- **LULU** (R function performing the co-occurrence-based post-clustering curation: flagging daughter OTUs and merging them into parents from the OTU table plus match list) — https://github.com/tobiasgf/lulu
- **BLASTn** (Generates the match list of internal OTU sequence similarities prior to curation (makeblastdb -parse_seqids -dbtype nucl; blastn -outfmt '6 qseqid sseqid pident' -qcov_hsp_perc 80 -perc_identity 84))
- **VSEARCH** (Alternative pairwise matcher for producing the match list (--usearch_global --self --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10); also used for initial OTU clustering)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); curated_result$curated_table
```

## Evaluation signals

- On the tutorial data with defaults, curated_count equals 459 and discarded_count equals 1966 (from 2425 initial OTUs over 130 samples).
- After curation, total OTUs fall below 564, the number of vascular plant species recorded in the field survey.
- Post-curation R2 of OTU-inferred vs field-survey site richness falls within the reported 0.56-0.71 range across the 20 method/level combinations.
- Total read counts per genus (e.g., Fagus, Calluna) are preserved after curation while daughter OTUs collapse onto their designated parent in the otu_map.
- Uncurated-vs-curated binary Bray-Curtis dissimilarity matrices are highly correlated, with Mantel r between 0.761 and 0.993 (p < 0.001).

## Limitations

- The 'cost of curation' is a small loss of real (rare, low-abundance) species, so curation is not suitable for identifying single rare species in community data sets.
- Validation was performed on vascular plants; relevance to other organism groups is inferred but not directly tested, and mock-community validation had not yet been carried out.
- A match list produced with a minimum match % (e.g., 84%) limits the effective dissimilarity threshold inside lulu: a lower threshold will have no effect.

## Evidence

- [readme] Co-occurrence-driven flagging of daughters as errors of more abundant parents: "By evaluating the co-occurence patterns of OTUs among samples **LULU** identifies OTUs that consistently satisfy some user selected criteria"
- [methods] OTU ordering step in the algorithm: "1. Sort the OTUs by decreasing occurrence, secondarily by total read count."
- [intro] Core daughter/parent merging mechanism: "The core mechanism is the identification and merging of ‘daughter’ OTUs with consistently co-occurring, sequence similar, but more abundant ‘parent’ OTUs"
- [discussion] Recommended overall pipeline placing curation in context: "we recommend an approach applying initial OTU definition with DADA2, subsequent clustering (with, e.g., VSEARCH) addressing the average intraspecific variation"
