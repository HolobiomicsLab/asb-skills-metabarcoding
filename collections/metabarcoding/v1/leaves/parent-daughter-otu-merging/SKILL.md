---
name: parent-daughter-otu-merging
description: Use when an OTU table (samples as columns, OTUs as rows) shows inflated
  richness from sequencing/PCR errors, and you have OTU centroid sequences from which
  a similarity match list can be produced for curation.
when_to_use_negative:
- When the goal is to identify single rare species in community data sets — LULU curation
  is not suitable for this and incurs a small loss of real rare species
- When no OTU centroid/representative sequences are available to produce a match list,
  since the algorithm requires sequence similarity to guide co-occurrence testing
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: LULU
  role: R function that takes the OTU table and matchlist and merges co-occurrence-based
    daughter errors into parents
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Produces the three-column match list of internal OTU matches with qcov_hsp_perc
    80 and perc_identity 84
- name: VSEARCH
  role: Alternative to BLASTn for generating the match list via --usearch_global with
    --id .84 --query_cov .9 --maxhits 10
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:dc1816b4a3cf20f5b2b8483b874184153c4de5846708f63e436cd587abe0c08a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - LULU
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# parent-daughter-otu-merging

## Summary

The LULU co-occurrence-based post-clustering curation step that identifies erroneous 'daughter' OTUs (sequence-similar, less abundant, consistently co-occurring with a more abundant 'parent') in an OTU table and merges their read counts into the parent to reduce artefactual OTU inflation.

## When to use

Use when an OTU table (samples as columns, OTUs as rows) shows inflated richness from sequencing/PCR errors, and you have OTU centroid sequences from which a similarity match list can be produced for curation.

## When NOT to use

- When the goal is to identify single rare species in community data sets — LULU curation is not suitable for this and incurs a small loss of real rare species
- When no OTU centroid/representative sequences are available to produce a match list, since the algorithm requires sequence similarity to guide co-occurrence testing

## Inputs

- OTU table (rows = OTUs with unique ids, columns = samples) e.g. otutable_test.txt with 2425 OTUs over 130 samples
- Fasta file of OTU representative/centroid sequences (e.g. centroids_test.txt) for building the match list
- Match list: three-column tab-separated file of query OTU id, matching OTU id, and percent similarity (e.g. match_list.txt from BLASTn or VSEARCH)

## Outputs

- curated_table.txt: OTU table with erroneous daughters' read counts merged into parents
- Curation result object with curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, and an otu_map (columns: total, spread, parent_id, curated, rank)

## How to apply

Sort OTUs by decreasing occurrence (spread), secondarily by total read count, then treat each OTU from top to bottom as a 'potential daughter'. From the match list (produced with BLASTn -outfmt '6 qseqid sseqid pident' -qcov_hsp_perc 80 -perc_identity 84, or VSEARCH --usearch_global --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10), select hits at or above minimum_match (84%), keep only those with occurrence ≥ the daughter as potential parents, and test each parent: relative co-occurrence must meet minimum_relative_cooccurence (default 0.95, i.e. 1 in 20 samples may lack parent presence) and the abundance ratio must meet minimum_ratio (default 1, minimum_ratio_type 'min' or 'avg'). Flag the first satisfying parent and mark the daughter as an error; otherwise keep it valid. Finally process the table from bottom to top, merging flagged errors (including errors-of-errors) into their ultimate parents so total read counts are preserved, and return the curated table with curation statistics. On the tutorial data (otutable_test.txt, 2425 OTUs over 130 samples) with defaults, the expected result is curated_count = 459 and discarded_count = 1966.

## Related tools

- **LULU** (R function that takes the OTU table and matchlist and merges co-occurrence-based daughter errors into parents) — https://github.com/tobiasgf/lulu
- **BLASTn** (Produces the three-column match list of internal OTU matches with qcov_hsp_perc 80 and perc_identity 84)
- **VSEARCH** (Alternative to BLASTn for generating the match list via --usearch_global with --id .84 --query_cov .9 --maxhits 10)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); curated_result$curated_table
```

## Evaluation signals

- With default parameters (minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95) on the tutorial data, curated_count equals 459 and discarded_count equals 1966
- Total read count is preserved: merged daughter reads are added to the parent, so column sums of the curated table match the original
- Every discarded OTU in the otu_map has a designated parent_id with occurrence ≥ its own and a match-list similarity ≥ minimum_match
- No retained daughter appears in samples absent from its parent at a rate below the 95% minimum_relative_cooccurence threshold
- The result object exposes curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, and otu_map with columns total, spread, parent_id, curated, rank

## Limitations

- The 'cost of curation' is a small loss of 'real species' (rare, low-abundance OTUs merged into parents)
- LULU curation may not be suitable for identifying single rare species in community data sets
- Validation was performed only on vascular plant ITS2 data; relevance to other organism groups is inferred but not directly tested
- Future validation on mock communities with known genetic content is still needed

## Evidence

- [intro] core mechanism: "The core mechanism is the identification and merging of ‘daughter’ OTUs with consistently co-occurring, sequence similar, but more abundant ‘parent’ OTUs across a multi-sample data set"
- [discussion] workflow requirements: [evidence span withheld — anchor status: not_found]
- [methods] merge step: "8. Process the OTU table from bottom to top, merging flagged errors with their parents to produce a curated OTU table."
- [readme] co-occurrence threshold: "If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%"
- [readme] README purpose: "By evaluating the co-occurence patterns of OTUs among samples **LULU** identifies OTUs that consistently satisfy some user selected criteria for being errors of more abundant OTUs and merges these."
