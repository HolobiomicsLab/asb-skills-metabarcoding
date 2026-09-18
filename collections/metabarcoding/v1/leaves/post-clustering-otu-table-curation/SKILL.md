---
name: post-clustering-otu-table-curation
description: Use when an OTU table (samples as columns, OTUs as rows with unique row
  names) over-inflates richness due to PCR/sequencing errors and rare artefactual
  OTUs, and you want reference-free post-clustering curation before biodiversity analyses.
when_to_use_negative:
- The goal is to identify single rare species in community datasets — LULU curation
  may not be suitable for identifying single rare species, and curation costs a small
  loss of real rare/low-abundance species
- You need de novo OTU inference itself — LULU requires an existing OTU table and
  a pre-computed match list; it does not perform initial clustering
- Validating against mock communities with known genetic content — this was explicitly
  flagged as untested future work for LULU
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3305
tools:
- name: LULU
  role: Core R function that flags and merges erroneous daughter OTUs using co-occurrence
    and abundance-ratio criteria, producing the curated OTU table
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: Can produce the OTU table being curated and the internal match list (--usearch_global
    --self --id .84 --iddef 1 --query_cov .9 --maxhits 10)
- name: SWARM
  role: One of the initial clustering methods whose OTU tables were curated with LULU
    in the validation (swarm has a Python experimental version in this repo)
  repo: https://github.com/frederic-mahe/swarm
- name: dbotu3
  role: Alternative curation/one-step clustering algorithm benchmarked against LULU
    (genetic dissimilarity 0.16, abundance criteria 0 and 10)
- name: BLASTn
  role: Reference tool for producing the match list (makeblastdb then blastn -outfmt
    '6 qseqid sseqid pident' -qcov_hsp_perc 80 -perc_identity 84)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:e81d1465a9423f177e206a0248d34ee38fdbd2ec35f8cf81e422a8c188bccd31
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
  - dbotu3
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# post-clustering-otu-table-curation

## Summary

Curate an already-clustered OTU table by merging erroneous 'daughter' OTUs into more abundant 'parent' OTUs using co-occurrence patterns and sequence similarity (LULU), improving alpha-diversity and taxonomic reliability without needing a reference database.

## When to use

Use when an OTU table (samples as columns, OTUs as rows with unique row names) over-inflates richness due to PCR/sequencing errors and rare artefactual OTUs, and you want reference-free post-clustering curation before biodiversity analyses.

## When NOT to use

- The goal is to identify single rare species in community datasets — LULU curation may not be suitable for identifying single rare species, and curation costs a small loss of real rare/low-abundance species
- You need de novo OTU inference itself — LULU requires an existing OTU table and a pre-computed match list; it does not perform initial clustering
- Validating against mock communities with known genetic content — this was explicitly flagged as untested future work for LULU

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names, integer read counts)
- FASTA file of representative/centroid sequences for each OTU (ids matching the OTU table)
- Match list: three-column tab-delimited file (query OTU id, matching OTU id, percent similarity), e.g. produced by BLASTn or VSEARCH

## Outputs

- Curated OTU table with erroneous daughter OTUs merged into parent OTUs
- Curation statistics (number of merged OTUs, curated/valid OTU counts) returned in the lulu result object

## How to apply

First produce the OTU table with any clustering method (e.g. VSEARCH, SWARM, CROP, DADA2), keeping the representative centroid/representative sequences whose OTU ids match the table. Generate a match list of internal OTU matches with BLASTn (e.g. -qcov_hsp_perc 80 -perc_identity 84) or VSEARCH (--self --id .84 --iddef 1 --query_cov .9 --maxhits 10) with three columns: query id, matching id, and percent similarity. In R, run the lulu function with the OTU table and match list; the algorithm sorts OTUs by decreasing occurrence then total read count, and flags a 'potential daughter' as an error of a 'potential parent' only if the parent is present in at least the minimum_relative_cooccurence (default 95%) of the daughter's samples AND the abundance ratio satisfies minimum_ratio (evaluated as 'min' or 'avg'). Flagged errors are merged into their parents from bottom to top, preserving total read count. Judge success by whether OTU counts drop while alpha-diversity, taxonomic redundancy, and best-match similarity metrics move closer to ground truth (e.g. field survey species richness).

## Related tools

- **LULU** (Core R function that flags and merges erroneous daughter OTUs using co-occurrence and abundance-ratio criteria, producing the curated OTU table) — https://github.com/tobiasgf/lulu
- **VSEARCH** (Can produce the OTU table being curated and the internal match list (--usearch_global --self --id .84 --iddef 1 --query_cov .9 --maxhits 10))
- **SWARM** (One of the initial clustering methods whose OTU tables were curated with LULU in the validation (swarm has a Python experimental version in this repo)) — https://github.com/frederic-mahe/swarm
- **dbotu3** (Alternative curation/one-step clustering algorithm benchmarked against LULU (genetic dissimilarity 0.16, abundance criteria 0 and 10))
- **BLASTn** (Reference tool for producing the match list (makeblastdb then blastn -outfmt '6 qseqid sseqid pident' -qcov_hsp_perc 80 -perc_identity 84))

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); curated_table <- curated_result$curated_table
```

## Evaluation signals

- The output table's row count (OTUs) is lower than the input table, and total read count per sample is preserved because errors are merged into parents rather than discarded
- Alpha-diversity estimates (e.g. species richness per site vs. the 6–93 species range of the field survey ground truth) move closer to survey values after curation
- Match list format is valid: three columns of query id, matching id, and percent similarity, with ids consistent between OTU table and sequence file
- Curation metrics (taxonomic redundancy, best GenBank match similarity, beta-diversity) improve relative to the uncurated table, as in Supplementary Figs 8–15
- The curated_result object exposes both curated_table and curation statistics; the curated table should be usable directly in downstream biodiversity analyses

## Limitations

- LULU curation may not be suitable for identifying single rare species in community data sets; the 'cost of curation' is a small loss of real (rare, low-abundance) species
- CROP OTU picking appeared suboptimal compared to the other methods investigated, which affects curation quality on such tables
- Validation was performed on vascular plants (ITS2) only; relevance to other organism groups is inferred but not directly tested, and validation on mock communities with known genetic content remains future work

## Evidence

- [intro] The core mechanism is the identification and merging of 'daughter' OTUs with consistently co-occurring, sequence similar, but more abundant 'parent' OTUs across a multi-sample data set: "The core mechanism is the identification and merging of ‘daughter’ OTUs with consistently co-occurring, sequence similar, but more abundant ‘parent’ OTUs across a multi-sample data set"
- [abstract] LULU curation consistently improves α-diversity estimates and other biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [readme] If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%: "If the number of samples where the 'potential parent' has a positive presence is below the `minimum_relative_cooccurence` (default 95%, meaning that 1 in 20 samples are allowed to have no parent"
- [discussion] The workflow (Fig. 2) requires (1) an OTU table ... and (2) a so-called match list—a list with the most similar OTUs ... produced prior to the LULU curation: [evidence span withheld — anchor status: not_found]
- [intro] We also tested the dbotu3 algorithm, both for 'one-step' clustering as intended, and as an alternative to LULU for post-clustering curation: "We also tested the dbotu3 algorithm, both for ‘one-step’ clustering as intended, and as an alternative to LULU for post-clustering curation"
