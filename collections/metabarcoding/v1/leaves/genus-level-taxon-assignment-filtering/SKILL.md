---
name: genus-level-taxon-assignment-filtering
description: Use when amplicon OTU tables from multiple clustering methods (VSEARCH,
  SWARM, CROP, DADA2) need genus-level error assessment using genera known to contain
  one abundant species as ground truth.
when_to_use_negative:
- 'When the goal is identifying single rare species: LULU curation may not be suitable
  because the ''cost of curation'' is a small loss of real, rare, low-abundance species'
- When the focal genus genuinely contains multiple co-occurring species, since collapse
  onto a dominant parent OTU would erase real diversity rather than remove errors
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0637
tools:
- name: LULU
  role: Performs the post-clustering curation whose otu_map and curated_table are
    inspected for focal-genus OTU collapse
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Produces the match list and supports assignment of OTU representative sequences
    to their best GenBank match
- name: VSEARCH
  role: One of the greedy clustering methods whose pre/post-curation OTU counts for
    Fagus and Calluna are compared; alternative match-list producer
- name: SWARM
  role: Clustering method included in the per-genus error/redundancy comparison
  repo: https://github.com/frederic-mahe/swarm
- name: DADA2
  role: Clustering method included in the per-genus error/redundancy comparison
- name: CROP
  role: Clustering method included in the per-genus error/redundancy comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:3a9127682f58b71626028a0297ce747d474bae069ec2de4e54085f7c99fb620c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Genus-level taxon assignment filtering

## Summary

Identifies OTUs assigned to specific focal genera (e.g. single-species genera like Fagus and Calluna) via best GenBank match, then quantifies erroneous/taxonomically redundant OTUs per clustering method and verifies that LULU curation collapses them onto dominant parent OTUs while preserving total read counts.

## When to use

Use when amplicon OTU tables from multiple clustering methods (VSEARCH, SWARM, CROP, DADA2) need genus-level error assessment using genera known to contain one abundant species as ground truth.

## When NOT to use

- When the goal is identifying single rare species: LULU curation may not be suitable because the 'cost of curation' is a small loss of real, rare, low-abundance species
- When the focal genus genuinely contains multiple co-occurring species, since collapse onto a dominant parent OTU would erase real diversity rather than remove errors

## Inputs

- Pre-curation OTU table per clustering method (samples as columns, OTUs as rows, unique OTU ids as row names)
- LULU curated_result object ($curated_table, $original_table, $otu_map)
- Match list of pairwise OTU similarities (query id, target id, percent identity) produced with BLASTn or VSEARCH
- Representative OTU centroid sequences in FASTA for best GenBank match assignment

## Outputs

- Per-genus CSV with OTU id, read counts before and after curation, best GenBank match, and parent_id from bottom-to-top error merging
- Plot data showing read-count homogenization (daughter-OTU read counts folded into designated more abundant parent OTUs)
- Post-curation per-genus OTU counts demonstrating realistic single-species diversity levels

## How to apply

Load in R the pre-curation OTU tables for each clustering method/level alongside the LULU-curated tables, accessing curated_result$original_table and curated_result$curated_table. Identify all OTUs whose best GenBank match is the focal genus (Fagus, Calluna) and extract per-sample and total read counts, parent_id, and curated status from curated_result$otu_map. Tabulate per-genus rows with OTU id, read counts before and after curation, best GenBank match, and parent assignment from LULU's bottom-to-top merging of flagged errors into more abundant parents. Validation is that post-curation total read counts per genus are preserved while the OTU count collapses onto the dominant parent, yielding realistic diversity for a genus containing one species.

## Related tools

- **LULU** (Performs the post-clustering curation whose otu_map and curated_table are inspected for focal-genus OTU collapse) — https://github.com/tobiasgf/lulu
- **BLASTn** (Produces the match list and supports assignment of OTU representative sequences to their best GenBank match)
- **VSEARCH** (One of the greedy clustering methods whose pre/post-curation OTU counts for Fagus and Calluna are compared; alternative match-list producer)
- **SWARM** (Clustering method included in the per-genus error/redundancy comparison) — https://github.com/frederic-mahe/swarm
- **DADA2** (Clustering method included in the per-genus error/redundancy comparison)
- **CROP** (Clustering method included in the per-genus error/redundancy comparison)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); otus_fagus <- rownames(curated_result$curated_table)[best_match == "Fagus"]
```

## Evaluation signals

- Post-curation total read counts per focal genus are preserved while the number of OTUs collapses onto the dominant parent OTU
- Number of OTUs assigned to Fagus or Calluna after curation approximates the realistic diversity of a genus with one abundant species
- Every collapsed daughter OTU in the curated output carries a parent_id consistent with LULU's bottom-to-top merging
- Curated table retains the same sample-column structure (130 sites/samples) as the original OTU table
- Homogenization plot shows daughter-OTU read counts being folded into their designated more abundant parent OTUs

## Limitations

- Curation carries a small loss of real (rare, low-abundance) species, so single rare species cannot be reliably identified after filtering
- LULU validation was performed only on vascular plants with ITS2 data from 130 Danish sites; relevance to other organism groups is inferred but not directly tested
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and β-diversity measures), which can confound genus-level comparisons

## Evidence

- [intro] For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank): "For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank)"
- [abstract] LULU consistently improves biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [discussion] Bottom-to-top merging preserves read counts: "After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top"
- [readme] LULU returns curated table and statistics: "The `lulu` function will return a curated OTU table and some statistics and information on the curation in the object `curated_result`."
- [discussion] Curation unsuitable for rare species: "LULU curation may thus not be suitable for identifying single rare species in community data sets"
