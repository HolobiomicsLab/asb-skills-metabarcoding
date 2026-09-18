---
name: otu-site-occupancy-filtering
description: Use when raw OTU count tables from eDNA metabarcoding (with PCR replicate
  detection data per sample) must be decontaminated before taxonomic grouping and
  downstream community analysis.
when_to_use_negative:
- Input is already a decontaminated/filtered feature or OTU table with no replicate-level
  detection information
- Only single-detection (non-replicated) samples are available, so per-OTU occurrence
  probability cannot be estimated from replicate detections
- The goal is taxonomic assignment rather than pre-grouping contamination removal;
  this filter operates as a quality-control step before taxonomic grouping
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: R
  role: Environment used to estimate per-OTU occurrence probability from PCR replicate
    detections and write the filtered OTU table
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:19d16008332551f5f99826d82defff520d685c0258e5d071f92228ddef6d11cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# otu-site-occupancy-filtering

## Summary

A contamination/negative-removal quality-control step for eDNA amplicon OTU tables in R: per-OTU occurrence probability is estimated from PCR replicate detections under a site-occupancy (binomial) framework, and OTUs with <80% estimated occurrence probability are dropped before taxonomic grouping.

## When to use

Use when raw OTU count tables from eDNA metabarcoding (with PCR replicate detection data per sample) must be decontaminated before taxonomic grouping and downstream community analysis.

## When NOT to use

- Input is already a decontaminated/filtered feature or OTU table with no replicate-level detection information
- Only single-detection (non-replicated) samples are available, so per-OTU occurrence probability cannot be estimated from replicate detections
- The goal is taxonomic assignment rather than pre-grouping contamination removal; this filter operates as a quality-control step before taxonomic grouping

## Inputs

- Raw OTU count table (samples x OTUs) from eDNA amplicon sequencing
- PCR replicate metadata (sample-by-replicate detection matrix)
- Taxonomic annotation columns associated with the OTU table

## Outputs

- filtered_otu_table.csv containing surviving OTU counts with original sample columns and taxonomic annotation columns preserved

## How to apply

Load the raw OTU count table together with the PCR replicate (sample-by-replicate) metadata in R. For each OTU, treat its presence/absence across PCR replicates as independent binomial draws and estimate the per-OTU probability of occurrence from the observed detection frequency under the site-occupancy model. Apply the decision threshold: remove every OTU whose estimated occurrence probability is below 0.8, which drops likely contaminant or sporadically detected sequences ('removal of negatives'). Write surviving OTU counts to filtered_otu_table.csv, preserving the original sample columns and taxonomic annotation columns; in the source study this filtered set (over 10^8 sequences) supported identification of 663 taxonomic groups at Family or higher rank.

## Related tools

- **R** (Environment used to estimate per-OTU occurrence probability from PCR replicate detections and write the filtered OTU table) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
Rscript -e 'occ <- colMeans(det_matrix); keep <- occ >= 0.8; write.csv(raw_otu[keep, ], "filtered_otu_table.csv", row.names = FALSE)'
```

## Evaluation signals

- No OTU in filtered_otu_table.csv has an estimated occurrence probability below 0.8
- Original sample columns and taxonomic annotation columns are preserved in the filtered output (schema check against raw table)
- Number of surviving OTUs is consistent with removal of negatives: dropped OTUs correspond to low/sporadic PCR replicate detection frequencies
- Filtered sequence set is of the order used downstream (source study: over 10^8 sequences yielding 663 taxonomic groups)
- Filter is applied before taxonomic grouping, so downstream community/network analysis consumes the filtered table only

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; primer biases, inhibitors and variable template abundances affect which taxa are detected
- The <80% cutoff can drop genuine but sporadically detected taxa along with contaminants
- eDNA cannot identify life history stages and may include sequences from dead or decaying biological matter, so filtering does not resolve biological interpretation issues

## Evidence

- [abstract] The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals.: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [discussion] No method, including this one, identifies all taxa in an ecosystem.: "no method—including this one—identifies all taxa in an ecosystem."
