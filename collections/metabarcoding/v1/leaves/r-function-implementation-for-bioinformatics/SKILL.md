---
name: r-function-implementation-for-bioinformatics
description: Use when an OTU table produced by any clustering algorithm (VSEARCH,
  SWARM, CROP, DADA2, UCLUST, QIIME) needs post-clustering curation in R to reduce
  erroneous OTUs before biodiversity analyses.
when_to_use_negative:
- The goal is to identify single rare species in community data sets — the article
  notes LULU curation 'may thus not be suitable for identifying single rare species'
- The input is raw sequence data rather than an OTU table — LULU requires an already-clustered
  OTU table plus a match list, not reads
- A reference-database-dependent curation is required — LULU is explicitly reference-database
  independent and cannot incorporate taxonomic reference information
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: LULU
  role: R package providing the co-occurrence-based post-clustering curation function
    being implemented/invoked; installed via devtools install_github('tobiasgf/lulu')
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Produces the match list input via makeblastdb (-parse_seqids -dbtype nucl)
    and blastn with -outfmt '6 qseqid sseqid pident', -qcov_hsp_perc 80, -perc_identity
    84
- name: VSEARCH
  role: 'Alternative match-list producer: --usearch_global --self --id .84 --iddef
    1 --userout match_list.txt --query_cov .9 --maxaccepts 0 --maxhits 10'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:dc99905ee022accaaa252e21586c4124c3e42caf6f855b3d5ddf8d0effd6a7b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - LULU
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# r-function-implementation-for-bioinformatics

## Summary

Implement and run an R function (following the LULU package pattern) that takes an OTU table and a match list and performs co-occurrence-based post-clustering curation, merging erroneous 'daughter' OTUs into more abundant 'parent' OTUs to yield more realistic biodiversity metrics.

## When to use

Use when an OTU table produced by any clustering algorithm (VSEARCH, SWARM, CROP, DADA2, UCLUST, QIIME) needs post-clustering curation in R to reduce erroneous OTUs before biodiversity analyses.

## When NOT to use

- The goal is to identify single rare species in community data sets — the article notes LULU curation 'may thus not be suitable for identifying single rare species'
- The input is raw sequence data rather than an OTU table — LULU requires an already-clustered OTU table plus a match list, not reads
- A reference-database-dependent curation is required — LULU is explicitly reference-database independent and cannot incorporate taxonomic reference information

## Inputs

- OTU table (tab-delimited, samples as columns, OTUs as rows, unique OTU ids as row names), e.g. otutable_test.txt with 2425 OTUs over 130 samples
- Match list (three columns: query OTU id, matching OTU id, percent similarity), e.g. match_list.txt from BLASTn or VSEARCH
- Representative OTU centroid sequences (FASTA, e.g. centroids_test.txt) used to build the match list

## Outputs

- Curated OTU table (curated_table.txt) with errors merged into parents and total read count preserved
- Curation summary object with curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, and an otu_map (columns total, spread, parent_id, curated parent/merged status, rank)

## How to apply

Write the R function to accept an OTU table (samples as columns, OTUs as rows, unique OTU ids as row names) and a three-column match list (query id, matching id, percent similarity) produced externally by BLASTn or VSEARCH. Sort OTUs by decreasing occurrence (spread), secondarily by total read count, then treat each OTU top-to-bottom as a 'potential daughter': select match-list hits at or above the dissimilarity threshold (e.g. minimum_match 84%), keep only hits with occurrence >= the daughter as 'potential parents', and flag the daughter as an error of the first parent that satisfies minimum_relative_cooccurence (default 0.95) and minimum_ratio (default 1, minimum_ratio_type 'min'); otherwise flag it valid. Process the flagged table bottom-to-top, merging read counts of flagged errors into their parents (allowing errors-of-errors to merge with ultimate parents) to preserve total read count. Return a result object containing curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, and an otu_map with parent_id and rank columns.

## Related tools

- **LULU** (R package providing the co-occurrence-based post-clustering curation function being implemented/invoked; installed via devtools install_github('tobiasgf/lulu')) — https://github.com/tobiasgf/lulu
- **BLASTn** (Produces the match list input via makeblastdb (-parse_seqids -dbtype nucl) and blastn with -outfmt '6 qseqid sseqid pident', -qcov_hsp_perc 80, -perc_identity 84)
- **VSEARCH** (Alternative match-list producer: --usearch_global --self --id .84 --iddef 1 --userout match_list.txt --query_cov .9 --maxaccepts 0 --maxhits 10)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name)
```

## Evaluation signals

- Run with defaults (minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95) on the tutorial data (otutable_test.txt + centroids_test.txt) and verify curated_count equals 459 and discarded_count equals 1966
- Curated table's total read count equals the original table's total read count (merging preserves reads)
- curated_count + discarded_count equals the original number of OTUs (2425 for the test data)
- Every discarded OTU in the otu_map has a parent_id whose occurrence is >= the discarded OTU's occurrence and whose sequence similarity is >= 84%
- Output object exposes the documented fields: curated_table, original_table, curated_count, discarded_count, curated_otus, discarded_otus, otu_map

## Limitations

- The 'cost of curation' is a small loss of real (rare, low-abundance) species
- Validation was performed only on vascular plant ITS2 data from 130 Danish field sites; relevance to other organism groups is inferred but not directly tested
- No validation yet on mock communities with known genetic content (flagged as future work)

## Evidence

- [intro] core mechanism identifies and merges daughter OTUs: [evidence span withheld — anchor status: not_found]
- [readme] match list requirement: "The required input of **LULU** is an OTU table and a corresponding matchlist with all the internal matches of OTUs."
- [methods] sort and co-occurrence criteria: "1. Sort the OTUs by decreasing occurrence, secondarily by total read count."
- [methods] co-occurrence threshold default: "If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%, meaning that 1 in 20"
- [methods] bottom-to-top merging: "8. Process the OTU table from bottom to top, merging flagged errors with their parents to produce a curated OTU table."
- [abstract] reference-database independence: "does not require a sequence reference database; thus, it represents a promising method for reliable biodiversity estimation"
- [discussion] read count preservation: "After identification of these extra OTUs, they can be merged with their parent OTUs in order to preserve the total read count and reduce the OTU number of the"
