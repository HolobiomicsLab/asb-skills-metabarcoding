---
name: amplicon-read-count-homogenization-analysis
description: Use when you have pre- and post-LULU OTU tables from multiple clustering
  methods (VSEARCH, SWARM, CROP, DADA2) and need to verify that curation homogenizes
  taxonomically redundant OTUs without losing total reads.
when_to_use_negative:
- When the goal is to detect single rare species — LULU curation may not be suitable
  for identifying single rare species in community data sets
- When you need a taxonomy-supervised clustering check — this skill is reference-independent;
  it only audits curation effects via read-count and parent-assignment structure
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: LULU
  role: Produces the curated table and otu_map (parent assignments) that the homogenization
    analysis compares against the original table
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the greedy clustering methods whose pre-curation OTU tables show the
    most pronounced read-count homogenization after LULU
- name: SWARM
  role: Greedy clustering method whose curation-induced read-count homogenization
    is analyzed alongside VSEARCH
  repo: https://github.com/frederic-mahe/swarm
- name: BLASTn
  role: Used to assign OTUs to species via best GenBank match for genus-level OTU
    identification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:67084312eb93f513fae936ef32f64a96fb3fb0e618433d0011a1cc40145c03e9
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# amplicon-read-count-homogenization-analysis

## Summary

Quantifies how LULU post-clustering curation folds erroneous daughter-OTU read counts into their more abundant parent OTUs, using single-species genera (e.g. Fagus and Calluna) as internal controls where post-curation OTU collapse to one dominant parent signals realistic diversity.

## When to use

Use when you have pre- and post-LULU OTU tables from multiple clustering methods (VSEARCH, SWARM, CROP, DADA2) and need to verify that curation homogenizes taxonomically redundant OTUs without losing total reads.

## When NOT to use

- When the goal is to detect single rare species — LULU curation may not be suitable for identifying single rare species in community data sets
- When you need a taxonomy-supervised clustering check — this skill is reference-independent; it only audits curation effects via read-count and parent-assignment structure

## Inputs

- Pre-curation OTU table (samples as columns, unique OTU ids as rows) per clustering method
- LULU-curated OTU table (curated_result$curated_table) per clustering method
- curated_result$otu_map giving each OTU's curated status and parent_id
- OTU representative sequences with best GenBank match taxonomic assignments

## Outputs

- Per-genus CSV table with OTU id, read counts before and after curation, best GenBank match, and parent assignment
- Plot data showing read-count homogenization (daughter-OTU reads folded into parent OTUs)
- Validation summary: post-curation total reads per genus preserved with OTUs collapsed onto the dominant parent

## How to apply

Load both the original (curated_result$original_table) and LULU-curated (curated_result$curated_table) OTU tables in R for each clustering method, and use curated_result$otu_map to get each OTU's parent_id and curated status. Identify OTUs assigned to single-species genera (e.g. Fagus, Calluna) via the best GenBank match of each OTU's representative sequence, then extract per-sample and total read counts before and after curation. Tabulate per-genus CSV rows with OTU id, read counts before/after curation, best GenBank match, and the parent assignment from LULU's bottom-to-top merging of errors into parents. Plot read-count homogenization, showing daughter-OTU read counts being folded into their designated parent OTUs. Validate that post-curation total read counts per genus are preserved while OTU counts collapse onto the dominant parent, confirming the homogenizing effect across methods.

## Related tools

- **LULU** (Produces the curated table and otu_map (parent assignments) that the homogenization analysis compares against the original table) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the greedy clustering methods whose pre-curation OTU tables show the most pronounced read-count homogenization after LULU)
- **SWARM** (Greedy clustering method whose curation-induced read-count homogenization is analyzed alongside VSEARCH) — https://github.com/frederic-mahe/swarm
- **BLASTn** (Used to assign OTUs to species via best GenBank match for genus-level OTU identification)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); dim(curated_result$original_table); dim(curated_result$curated_table); head(curated_result$otu_map)
```

## Evaluation signals

- Post-curation total read count per genus is preserved while the number of OTUs collapses onto the dominant parent OTU
- For single-species genera (Fagus, Calluna), curated diversity reaches realistic levels of ~1 dominant OTU per genus
- Every flagged daughter OTU in otu_map has a valid parent_id from the bottom-to-top merge, and no parent is its own daughter
- Read-count homogenization plots show daughter-OTU per-sample counts summing into the parent's counts across samples
- The effect is consistent across clustering methods, with the strongest collapse for the greedy methods VSEARCH and SWARM

## Limitations

- For genera with only one species, the number of erroneous OTUs was high for most methods, so pre-curation counts may be dominated by redundancy
- LULU curation may not be suitable for identifying single rare species in community data sets; the 'cost of curation' is a small loss of 'real species'
- Homogenization is inferred from best GenBank match taxonomy; incomplete reference databases can mislabel OTUs and distort genus-level tallies

## Evidence

- [results] For genera with only one species represented (such as Fagus and Calluna), the number of erroneous or at least taxonomically redundant OTUs was high for most methods, and LULU curation resulted in realistic levels of diversity: [evidence span withheld — anchor status: not_found]
- [discussion] After identification of these extra OTUs, they can be merged with their parent OTUs in order to preserve the total read count and reduce the OTU number of the table: "After identification of these extra OTUs, they can be merged with their parent OTUs in order to preserve the total read count and reduce the OTU number of the table"
- [discussion] a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top: "After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top"
- [readme] The lulu function will return a curated OTU table and some statistics and information on the curation in the object `curated_result`.: "The `lulu` function will return a curated OTU table and some statistics and information on the curation in the object `curated_result`."
- [discussion] The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs: "The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs"
