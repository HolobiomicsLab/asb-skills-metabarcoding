---
name: amplicon-count-table-processing
description: Use when a raw sample-by-OTU count table from multi-replicate eDNA amplicon
  sequencing (with PCR replicate metadata) needs decontamination before taxonomic
  grouping or community/network analysis.
when_to_use_negative:
- The input is already a decontaminated/filtered OTU table or a taxonomic-group-level
  abundance table — re-applying the replicate-based binomial filter would discard
  legitimate rare detections.
- The data lack PCR replicate structure (single detection per sample), since the occurrence-probability
  estimate depends on detections across independent PCR replicates.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: R
  role: Environment used to load the raw OTU table and replicate metadata, estimate
    per-OTU occurrence probability from replicate detections, apply the <80% cutoff,
    and write filtered_otu_table.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:22ca947cd37a06daf27d9c2b7d82a87bc6fb60574e1d049604be8f5ac9130c33
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 3
---

# amplicon-count-table-processing

## Summary

A quality-control skill that removes likely contaminant OTUs from raw eDNA amplicon count tables by modeling PCR replicate detections as independent binomial draws and dropping OTUs with <80% estimated occurrence probability, so downstream taxonomic grouping and community analysis rest on a decontaminated sequence set.

## When to use

Use when a raw sample-by-OTU count table from multi-replicate eDNA amplicon sequencing (with PCR replicate metadata) needs decontamination before taxonomic grouping or community/network analysis.

## When NOT to use

- The input is already a decontaminated/filtered OTU table or a taxonomic-group-level abundance table — re-applying the replicate-based binomial filter would discard legitimate rare detections.
- The data lack PCR replicate structure (single detection per sample), since the occurrence-probability estimate depends on detections across independent PCR replicates.

## Inputs

- raw OTU count table (sample-by-OTU matrix) from eDNA multilocus amplicon sequencing
- PCR replicate metadata mapping samples to replicates
- taxonomic annotation columns associated with the OTU table

## Outputs

- filtered_otu_table.csv containing only OTUs with >=0.8 estimated occurrence probability, with original sample columns and taxonomic annotations preserved
- filtered sequence set used to identify 663 taxonomic groups for downstream community network analysis

## How to apply

Load the raw OTU count table together with the PCR replicate (sample-by-replicate) metadata in R. For each OTU, treat its presence/absence across PCR replicates as independent binomial draws and estimate the per-OTU probability of occurrence from the observed detection frequency under a site-occupancy framework. Apply the 0.8 threshold: remove every OTU whose estimated occurrence probability is below 80%, which drops likely contaminants or sporadically detected sequences. Write the surviving OTU counts to filtered_otu_table.csv, preserving the original sample columns and taxonomic annotation columns. In the source study this 'removal of negatives' step yielded the filtered sequence set (over 10^8 sequences) from which 663 taxonomic groups (Family or higher) were identified, so correct application should reproduce a count table of comparable structure feeding the downstream network analysis.

## Related tools

- **R** (Environment used to load the raw OTU table and replicate metadata, estimate per-OTU occurrence probability from replicate detections, apply the <80% cutoff, and write filtered_otu_table.csv)

## Examples

```
write.csv(otu_survivors, "filtered_otu_table.csv", row.names = FALSE)
```

## Evaluation signals

- Filtered table contains only OTUs whose estimated occurrence probability is >= 0.8 under the site-occupancy/binomial model; all OTUs below the cutoff are absent.
- filtered_otu_table.csv preserves the original sample columns and taxonomic annotation columns of the input count table.
- The filtered sequence set totals over 10^8 sequences and supports identification of 663 taxonomic groups (Family or higher), matching the published scale.
- Per-OTU occurrence probabilities are computed from PCR replicate detection frequencies, so OTUs detected in few replicates map to low probabilities and are the ones removed.
- The step is applied before taxonomic grouping and downstream community/network analysis, not after.

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; the filter may remove real low-occurrence taxa along with contaminants.
- Amplification is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, which affect detection frequencies feeding the binomial model.
- eDNA does not allow identification of life history stages and may include sequences from dead or decaying biological matter, so a passing OTU is not necessarily from a living local organism.

## Evidence

- [abstract] The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals.: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [other] treats PCR replicate detections as independent binomial draws, estimates per-OTU occurrence probability under a site-occupancy framework: [evidence span withheld — anchor status: not_found]
- [discussion] Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
