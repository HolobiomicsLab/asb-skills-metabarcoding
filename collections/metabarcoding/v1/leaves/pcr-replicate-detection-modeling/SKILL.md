---
name: pcr-replicate-detection-modeling
description: Use when a raw eDNA amplicon OTU count table has PCR replicate-level
  detection records (sample-by-replicate) and contaminant/sporadic OTUs must be removed
  prior to taxonomic grouping and downstream community analysis.
when_to_use_negative:
- Input lacks PCR replicate-level detection records (only a single detection observation
  per OTU per sample), so a binomial occurrence probability cannot be estimated under
  the site-occupancy framework.
- Input is already a decontaminated/filtered OTU table (e.g., blanks already removed
  by another 'removal of negatives' step); reapplying the <0.8 occupancy filter would
  risk double-filtering true detections.
- The analysis goal is to retain rare or sporadically detected taxa (e.g., low-biomass
  species at the detection limit), since the <0.8 occurrence-probability cutoff will
  drop exactly such sequences.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: R
  role: Used to load the raw OTU count table and replicate metadata, fit the per-OTU
    binomial/site-occupancy occurrence-probability estimates, apply the <0.8 filter,
    and write filtered_otu_table.csv
  repo: https://github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:8ed87044814bb7a386954c5602b2061283ce79b58144ef42caec2fce4b1c901a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# PCR-replicate detection modeling (site-occupancy decontamination filter)

## Summary

A R-based quality-control skill that treats PCR replicate detections of each OTU as independent binomial draws, estimates a per-OTU occurrence probability under a site-occupancy framework, and removes OTUs with <80% estimated occurrence probability to eliminate contaminant or sporadically detected sequences before taxonomic grouping.

## When to use

Use when a raw eDNA amplicon OTU count table has PCR replicate-level detection records (sample-by-replicate) and contaminant/sporadic OTUs must be removed prior to taxonomic grouping and downstream community analysis.

## When NOT to use

- Input lacks PCR replicate-level detection records (only a single detection observation per OTU per sample), so a binomial occurrence probability cannot be estimated under the site-occupancy framework.
- Input is already a decontaminated/filtered OTU table (e.g., blanks already removed by another 'removal of negatives' step); reapplying the <0.8 occupancy filter would risk double-filtering true detections.
- The analysis goal is to retain rare or sporadically detected taxa (e.g., low-biomass species at the detection limit), since the <0.8 occurrence-probability cutoff will drop exactly such sequences.

## Inputs

- Raw OTU count table (samples x OTUs) from eDNA amplicon sequencing
- PCR replicate metadata (sample-by-replicate detection records)
- Taxonomic annotation columns accompanying the OTU table

## Outputs

- filtered_otu_table.csv with surviving OTU counts, original sample columns, and taxonomic annotation columns
- Set of dropped OTUs whose estimated occurrence probability was <0.8

## How to apply

Load the raw OTU count table together with the PCR replicate (sample-by-replicate) metadata in R. For each OTU, model its presence/absence across PCR replicates as independent draws from a binomial distribution and estimate the per-OTU probability of occurrence from the observed detection frequency, following the site-occupancy ('removal of negatives') framework used in the Monterey Bay 18-month eDNA time-series. Apply the 0.8 threshold: drop every OTU whose estimated occurrence probability is below 0.8, treating these as likely contaminants or sporadically detected sequences. Write the surviving OTU counts to filtered_otu_table.csv, preserving the original sample columns and taxonomic annotation columns so the filtered set (here over 10^8 sequences) can feed taxonomic grouping that yields 663 taxonomic groups.

## Related tools

- **R** (Used to load the raw OTU count table and replicate metadata, fit the per-OTU binomial/site-occupancy occurrence-probability estimates, apply the <0.8 filter, and write filtered_otu_table.csv) — https://github.com/marinebon/eDNA_microbes_whales

## Examples

```
otu_keep <- otu[otu$occurrence_prob >= 0.8, ]; write.csv(otu_keep, "filtered_otu_table.csv", row.names = FALSE)
```

## Evaluation signals

- Schema check: filtered_otu_table.csv retains the original sample columns and taxonomic annotation columns present in the raw OTU table.
- Invariant: every dropped OTU has an estimated occurrence probability <0.8 under the binomial replicate-detection model; every retained OTU is >=0.8.
- Scale check: the filtered sequence set contains more than 10^8 sequences and supports identification of 663 taxonomic groups (at Family or higher taxonomic rank).
- Reproducibility: rerunning the filter on the same raw OTU table and replicate metadata yields an identical set of surviving OTUs.
- Contamination sanity check: OTUs detected only in a small fraction of PCR replicates (low observed detection frequency) are absent from the filtered table.

## Limitations

- Amplification is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, which can distort replicate detection frequencies and hence the estimated occurrence probabilities.
- The <0.8 cutoff will remove true but sporadically detected taxa, and no method — including this one — identifies all taxa in an ecosystem.
- Filtering is performed on OTU detections at replicate level only; it does not resolve species-level assignment confidence, which decreases at lower taxonomic ranks.

## Evidence

- [readme] README decontamination/QC scope: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [abstract] Dataset scale and taxonomic groups produced from the filtered sequences: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [abstract] Origin of the amplicon data being quality-filtered: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [discussion] Amplification caveat affecting replicate detections: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
