---
name: binomial-occupancy-probability-estimation
description: Use when a raw OTU/ASV count table from an eDNA amplicon study includes
  PCR replicate metadata, and contamination or sporadic detection must be removed
  before taxonomic grouping and downstream community analyses.
when_to_use_negative:
- Input is already a quality-filtered feature/OTU table or lacks PCR replicate information
  needed for the binomial detection model
- Data are shotgun metagenomic or quantitative abundance data rather than multilocus
  amplicon presence/absence detections, where occupancy-based removal of negatives
  is inappropriate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Used to load the OTU table and replicate metadata, fit the per-OTU binomial/occupancy
    occurrence estimates, apply the <80% cutoff, and write filtered_otu_table.csv
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:a54ff628337c3131921d6768df8fe789d6e514f8d358b12146e65fb6af5ccd72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# binomial-occupancy-probability-estimation

## Summary

A site-occupancy decontamination filter for eDNA amplicon data that treats per-OTU detections across PCR replicates as independent binomial draws, estimates each OTU's occurrence probability, and drops OTUs estimated at <80% to remove contaminant/sporadic sequences.

## When to use

Use when a raw OTU/ASV count table from an eDNA amplicon study includes PCR replicate metadata, and contamination or sporadic detection must be removed before taxonomic grouping and downstream community analyses.

## When NOT to use

- Input is already a quality-filtered feature/OTU table or lacks PCR replicate information needed for the binomial detection model
- Data are shotgun metagenomic or quantitative abundance data rather than multilocus amplicon presence/absence detections, where occupancy-based removal of negatives is inappropriate

## Inputs

- Raw OTU count table (samples x OTUs with taxonomic annotation columns)
- PCR replicate metadata (sample-by-replicate detection matrix)
- Occurrence-probability threshold (0.8 in the source study)

## Outputs

- filtered_otu_table.csv: OTU counts for OTUs with estimated occurrence probability >= 0.8, with sample columns and taxonomic annotations preserved
- Per-OTU estimated occurrence probabilities

## How to apply

Load the raw OTU count table together with the sample-by-replicate PCR metadata in R. For each OTU, treat its presence/absence across PCR replicates as independent binomial draws and estimate the per-OTU probability of occurrence from the observed detection frequency under a site-occupancy framework. Remove every OTU whose estimated occurrence probability is below the 0.8 cutoff, treating these as likely contaminants or sporadically detected sequences ('removal of negatives'). Write surviving OTU counts to filtered_otu_table.csv, preserving the original sample columns and taxonomic annotation columns. In the source study this quality-control step preceded taxonomic grouping and yielded a filtered sequence set (over 10^8 sequences) that resolved 663 taxonomic groups (Family or higher) in an 18-month Monterey Bay seawater time-series.

## Related tools

- **R** (Used to load the OTU table and replicate metadata, fit the per-OTU binomial/occupancy occurrence estimates, apply the <80% cutoff, and write filtered_otu_table.csv) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
Rscript occupancy_filter.R raw_otu_counts.csv pcr_replicate_metadata.csv 0.8 filtered_otu_table.csv
```

## Evaluation signals

- Every OTU retained in the output has an estimated occurrence probability >= 0.8 under the binomial/occupancy model
- filtered_otu_table.csv preserves the original sample columns and taxonomic annotation columns without reordering
- No OTU absent from all but sporadic PCR replicates survives the filter (contaminants and 'negatives' removed)
- Downstream taxonomic grouping should be reproducible, with the filtered set supporting the reported scale of results (e.g. 663 taxonomic groups from >10^8 sequences in the source study)
- Output row count is <= input OTU count, with removed rows traceable to low detection frequencies across replicates

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; filtering discards OTUs whose detections may reflect real but rare occurrence
- Amplification is subject to primer biases, inhibitors, and variable template abundances, which affect the detection frequencies feeding the occupancy estimates
- Estimates depend on PCR replicates being independent binomial draws; eDNA cannot resolve life-history stages and may include sequences from dead or decaying biological matter

## Evidence

- [other] The decontamination filter operates as a quality-control step ('removal of negatives') that treats PCR replicate detections as independent binomial draws: [evidence span withheld — anchor status: not_found]
- [other] eliminates OTUs with <80% estimated occurrence probability, yielding the filtered sequence set: [evidence span withheld — anchor status: not_found]
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
