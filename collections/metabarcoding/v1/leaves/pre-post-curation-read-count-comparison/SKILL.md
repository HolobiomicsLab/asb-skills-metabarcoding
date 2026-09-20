---
name: pre-post-curation-read-count-comparison
description: Use when comparing pre- and post-LULU-curation OTU tables to assess how
  many erroneous or taxonomically redundant OTUs (e.g., for single-species genera
  like Fagus or Calluna) collapse onto parent OTUs and whether read counts are conserved.
when_to_use_negative:
- When the goal is to identify single rare species in a community dataset — LULU curation
  may discard rare but real OTUs (the 'cost of curation'), so a pre/post comparison
  would report their loss as curation effect rather than error removal
- When the input is not an OTU table paired with an internal match list (e.g., raw
  FASTQ reads or ASV data without representative sequences) — LULU requires both an
  OTU table and a match list produced by BLASTn or VSEARCH
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: LULU
  role: Performs the post-clustering curation whose pre/post effect is being compared;
    returns curated_result with original_table, curated_table, and otu_map for parent-daughter
    read-count tracing
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Produces the internal match list of OTU-to-OTU percent similarities that drives
    LULU's parent assignment; also used to assign OTUs to best GenBank matches
- name: VSEARCH
  role: Alternative producer of the match list (--usearch_global --self --id .84 --iddef
    1) and one of the initial clustering methods whose OTU tables are compared before
    and after curation
- name: SWARM
  role: Initial greedy clustering method producing OTU tables whose pre/post-curation
    read-count distribution is compared
  repo: https://github.com/frederic-mahe/swarm
- name: DADA2
  role: Initial OTU-definition method producing OTU tables whose pre/post-curation
    read-count distribution is compared
- name: CROP
  role: Initial clustering method producing OTU tables whose pre/post-curation read-count
    distribution is compared (its OTU picking appeared suboptimal)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:4c8ba7a477f2bd0390b287b06be9d5c8f0a43dda8926653ee02025fcdf6530f7
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
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pre-post-curation-read-count-comparison

## Summary

Quantifies the homogenizing effect of LULU post-clustering curation by tabulating per-OTU and per-genus read counts in the original versus curated OTU tables, verifying that daughter-OTU reads are folded into their designated more abundant parent OTUs while total read counts per taxon are preserved.

## When to use

Use when comparing pre- and post-LULU-curation OTU tables to assess how many erroneous or taxonomically redundant OTUs (e.g., for single-species genera like Fagus or Calluna) collapse onto parent OTUs and whether read counts are conserved.

## When NOT to use

- When the goal is to identify single rare species in a community dataset — LULU curation may discard rare but real OTUs (the 'cost of curation'), so a pre/post comparison would report their loss as curation effect rather than error removal
- When the input is not an OTU table paired with an internal match list (e.g., raw FASTQ reads or ASV data without representative sequences) — LULU requires both an OTU table and a match list produced by BLASTn or VSEARCH

## Inputs

- Pre-curation OTU table (samples as columns, unique OTU ids as row names) for each clustering method/level
- LULU curated_result object containing original_table, curated_table, and otu_map (with parent_id and curated status)
- Match list (three columns: query OTU id, matching OTU id, percent similarity) produced by BLASTn or VSEARCH
- Representative OTU sequences (FASTA) used for BLASTn/VSEARCH matching and best GenBank match assignment

## Outputs

- Per-genus CSV rows with OTU id, read counts before and after curation, best GenBank match, and parent assignment
- Validation summary confirming post-curation total read counts per genus are preserved while OTU number collapses onto the dominant parent
- Plot data showing read-count homogenization (daughter-OTU read counts folded into their designated more abundant parent OTUs)

## How to apply

Load in R the pre-curation OTU tables for each clustering method (VSEARCH, SWARM, CROP, DADA2) and the corresponding LULU-curated tables via curated_result$original_table and curated_result$curated_table. Identify OTUs of interest (e.g., all OTUs assigned to Fagus or Calluna by the best GenBank match of each OTU's representative sequence) and extract per-sample and total read counts, curated status, and parent_id from curated_result$otu_map. Tabulate per-genus rows with OTU id, read counts before and after curation, best GenBank match, and the parent assignment from the bottom-to-top merging of errors into parents. Validate that post-curation total read counts per genus are preserved while the OTU number collapses onto the dominant parent, yielding realistic diversity levels for genera containing one abundant species; plot the read-count homogenization showing daughter reads being folded into parents.

## Related tools

- **LULU** (Performs the post-clustering curation whose pre/post effect is being compared; returns curated_result with original_table, curated_table, and otu_map for parent-daughter read-count tracing) — https://github.com/tobiasgf/lulu
- **BLASTn** (Produces the internal match list of OTU-to-OTU percent similarities that drives LULU's parent assignment; also used to assign OTUs to best GenBank matches)
- **VSEARCH** (Alternative producer of the match list (--usearch_global --self --id .84 --iddef 1) and one of the initial clustering methods whose OTU tables are compared before and after curation)
- **SWARM** (Initial greedy clustering method producing OTU tables whose pre/post-curation read-count distribution is compared) — https://github.com/frederic-mahe/swarm
- **DADA2** (Initial OTU-definition method producing OTU tables whose pre/post-curation read-count distribution is compared)
- **CROP** (Initial clustering method producing OTU tables whose pre/post-curation read-count distribution is compared (its OTU picking appeared suboptimal))

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); orig <- curated_result$original_table; cur <- curated_result$curated_table
```

## Evaluation signals

- Post-curation total read counts per genus (e.g., Fagus, Calluna) are preserved relative to the pre-curation table, even though the number of OTUs collapses
- The number of OTUs assigned to a single-species genus collapses onto the dominant parent OTU, yielding realistic diversity levels for that genus
- Each merged daughter OTU in the curated table maps to a parent_id in curated_result$otu_map that was a valid 'hit' in the match list within the dissimilarity threshold
- Summed read counts of flagged daughters plus parent equal the parent's post-curation total in curated_table, demonstrating bottom-to-top merging
- Curated OTU counts for greedy methods (VSEARCH, SWARM) show the most pronounced reduction, consistent with their initially highest OTU numbers

## Limitations

- LULU curation may not be suitable for identifying single rare species in community data sets; the 'cost of curation' is a small loss of 'real' (rare, low-abundance) species
- Merging assumes daughters are artefacts based on co-occurrence and similarity; incomplete reference databases make it hard to separate errors from true OTUs, so some real variants may be merged
- CROP's OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and beta-diversity), so pre/post comparisons for CROP tables may reflect clustering artefacts rather than curation effect alone

## Evidence

- [discussion] After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top: "After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top"
- [intro] Core mechanism of daughter-parent merging: "The core mechanism is the identification and merging of ‘daughter’ OTUs with consistently co-occurring, sequence similar, but more abundant ‘parent’ OTUs across a multi-sample data set"
- [methods] Bottom-to-top merging step in the algorithm: "8. Process the OTU table from bottom to top, merging flagged errors with their parents to produce a curated OTU table."
- [readme] LULU identifies errors by co-occurrence patterns and merges them: "By evaluating the co-occurence patterns of OTUs among samples **LULU** identifies OTUs that consistently satisfy some user selected criteria for being errors of more abundant OTUs and merges these."
- [readme] lulu function returns curated table and statistics: "The `lulu` function will return a curated OTU table and some statistics and information on the curation in the object `curated_result`."
