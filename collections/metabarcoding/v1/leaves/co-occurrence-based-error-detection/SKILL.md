---
name: co-occurrence-based-error-detection
description: Use when an OTU/ASV table contains inflated richness from PCR/sequencing
  errors or intraspecific variation, and you need reference-free post-clustering curation
  that preserves rare but real OTUs.
when_to_use_negative:
- The goal is to identify single rare species in community data sets — LULU curation
  may not be suitable for this
- A reference-database-driven error correction is required — LULU is independent of
  a reference database and relies on co-occurrence rather than external taxonomic
  references
- The input is raw reads rather than an OTU table with a matching centroid sequence
  file and match list — the workflow requires an existing OTU table plus representative
  sequences
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: LULU
  role: R function that takes the OTU table and match list, flags co-occurring erroneous
    daughters by co-occurrence and abundance-ratio criteria, and merges them into
    parents
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Produces the three-column match list of internal OTU matches with -qcov_hsp_perc
    80 and -perc_identity 84
- name: VSEARCH
  role: Alternative to BLASTn for producing the match list via --usearch_global with
    --id .84 --query_cov .9 --maxaccepts 0 --maxhits 10
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:2ff9168dbcc798504a32992c0dc46560b32535b1264d099ce0afa0954652e52a
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# co-occurrence-based-error-detection

## Summary

The LULU algorithm identifies erroneous 'daughter' OTUs in an OTU table by evaluating co-occurrence patterns among samples, guided by pairwise sequence similarity from a match list, and merges flagged errors into more abundant 'parent' OTUs to produce a curated table with more realistic α-diversity estimates.

## When to use

Use when an OTU/ASV table contains inflated richness from PCR/sequencing errors or intraspecific variation, and you need reference-free post-clustering curation that preserves rare but real OTUs.

## When NOT to use

- The goal is to identify single rare species in community data sets — LULU curation may not be suitable for this
- A reference-database-driven error correction is required — LULU is independent of a reference database and relies on co-occurrence rather than external taxonomic references
- The input is raw reads rather than an OTU table with a matching centroid sequence file and match list — the workflow requires an existing OTU table plus representative sequences

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names; e.g. otutable_test.txt with 2425 OTUs over 130 samples)
- FASTA file of representative OTU centroid sequences (e.g. centroids_test.txt)
- Match list: three-column tab-delimited file with query OTU id, matching OTU id, and percent pairwise similarity (produced by BLASTn or VSEARCH)

## Outputs

- Curated OTU table (curated_table.txt) with erroneous OTUs merged into parents
- Curation summary object containing curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, and an otu_map (columns: total, spread, parent_id, curated, rank)

## How to apply

Produce an OTU table (samples as columns, OTUs as rows, unique OTU ids as row names) and a match list with BLASTn (`makeblastdb -parse_seqids -dbtype nucl`, then `blastn -outfmt '6 qseqid sseqid pident' -qcov_hsp_perc 80 -perc_identity 84`) or VSEARCH (`--usearch_global --self --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10`). In R, sort OTUs by decreasing occurrence, secondarily by total read count; treat each OTU top-to-bottom as a 'potential daughter', select match-list 'hits' at or above the dissimilarity threshold (minimum_match 84%), keep only 'potential parents' with occurrence at or above the daughter's, and test each parent: relative co-occurrence must meet minimum_relative_cooccurence (default 0.95) and the observed abundance ratio must meet minimum_ratio (default 1, minimum_ratio_type 'min'). Flag the daughter as an error of the first satisfying parent, otherwise flag it valid; finally process the table from bottom to top, merging read counts of flagged errors with their designated parents (allowing errors-of-errors to merge with ultimate parents) to preserve total read count. Rationale: the core mechanism assumes that sequence-similar OTUs whose occurrence is consistently explained by co-occurrence with a more abundant parent are artefacts, and merging them retains true α-diversity without discarding rare real OTUs.

## Related tools

- **LULU** (R function that takes the OTU table and match list, flags co-occurring erroneous daughters by co-occurrence and abundance-ratio criteria, and merges them into parents) — https://github.com/tobiasgf/lulu
- **BLASTn** (Produces the three-column match list of internal OTU matches with -qcov_hsp_perc 80 and -perc_identity 84)
- **VSEARCH** (Alternative to BLASTn for producing the match list via --usearch_global with --id .84 --query_cov .9 --maxaccepts 0 --maxhits 10)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name)
```

## Evaluation signals

- Run with defaults (minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95) on the tutorial data and verify curated_count equals 459 and discarded_count equals 1966
- Total read count is preserved after curation (errors are merged, not discarded)
- The output object contains the expected fields: curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, otu_map
- Match list is three columns (query OTU id, matching OTU id, percent similarity) and OTU ids match the OTU table row names
- Every discarded OTU in the otu_map has a designated parent_id whose occurrence is at least that of the daughter

## Limitations

- The 'cost of curation' is a small loss of 'real' (rare, low-abundance) species
- LULU curation may not be suitable for identifying single rare species in community data sets
- Validation was on vascular plant ITS2 soil data (130 Danish field sites); relevance to other organism groups is inferred but not directly tested, and future validation on mock communities with known genetic content will be valuable

## Evidence

- [intro] The core mechanism is the identification and merging of 'daughter' OTUs with consistently co-occurring, sequence similar, but more abundant 'parent' OTUs across a multi-sample data set: "The core mechanism is the identification and merging of ‘daughter’ OTUs with consistently co-occurring, sequence similar, but more abundant ‘parent’ OTUs across a multi-sample data set"
- [methods] Sort the OTUs by decreasing occurrence, secondarily by total read count.: "1. Sort the OTUs by decreasing occurrence, secondarily by total read count."
- [discussion] A new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top: "After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top"
- [readme] blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta: "blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta"
- [abstract] LULU curation consistently improves α-diversity estimates and other biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
