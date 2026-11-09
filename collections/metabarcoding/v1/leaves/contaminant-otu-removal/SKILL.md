---
name: contaminant-otu-removal
description: Use when a raw eDNA amplicon OTU count table includes PCR replicate-level
  detections and suspected contaminant or sporadically detected OTUs must be removed
  via a site-occupancy occurrence-probability filter before taxonomic grouping.
when_to_use_negative:
- The OTU table has no PCR replicate structure — without sample-by-replicate detections
  the binomial/site-occupancy occurrence probability cannot be estimated.
- Input is already taxonomically grouped (e.g., the 663 Family-or-higher groups) or
  is a post-filter sequence set; this QC step operates on raw OTU counts before grouping.
- Data are shotgun metagenomes or other non-replicated amplicon calls rather than
  PCR-replicated eDNA amplicon detections, where the replicate-based occurrence model
  does not apply.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Used to load the raw OTU count table and PCR replicate metadata, estimate
    per-OTU occurrence probabilities from replicate detections as binomial draws,
    apply the <0.8 removal threshold, and write filtered_otu_table.csv
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:0c85d76324f2bde493943124be11196f4f591b51031a80a7bc8a74b9c8974d6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# contaminant-otu-removal

## Summary

A site-occupancy-based decontamination QC filter for eDNA amplicon data: PCR replicate detections are treated as independent binomial draws to estimate each OTU's occurrence probability, and OTUs with <80% estimated occurrence probability are eliminated. This 'removal of negatives' step ensures only reproducibly detected taxa feed taxonomic grouping and downstream community analysis.

## When to use

Use when a raw eDNA amplicon OTU count table includes PCR replicate-level detections and suspected contaminant or sporadically detected OTUs must be removed via a site-occupancy occurrence-probability filter before taxonomic grouping.

## When NOT to use

- The OTU table has no PCR replicate structure — without sample-by-replicate detections the binomial/site-occupancy occurrence probability cannot be estimated.
- Input is already taxonomically grouped (e.g., the 663 Family-or-higher groups) or is a post-filter sequence set; this QC step operates on raw OTU counts before grouping.
- Data are shotgun metagenomes or other non-replicated amplicon calls rather than PCR-replicated eDNA amplicon detections, where the replicate-based occurrence model does not apply.

## Inputs

- Raw OTU count table (OTUs x samples) from eDNA multilocus amplicon sequencing, with PCR replicate-level detections
- PCR replicate metadata: sample-by-replicate table mapping each sample to its PCR replicates
- Taxonomic annotation columns accompanying the OTU table

## Outputs

- filtered_otu_table.csv — surviving OTU counts (estimated occurrence probability >= 0.8), preserving the original sample columns and taxonomic annotation columns
- Filtered sequence set (over 10^8 sequences) used downstream to identify 663 taxonomic groups (Family or higher)

## How to apply

In R, load the raw OTU count table together with the sample-by-replicate PCR replicate metadata. For each OTU, treat its presence/absence across PCR replicates as independent draws from a binomial distribution and estimate the per-OTU probability of occurrence from the observed detection frequency under a site-occupancy framework. Apply the decision threshold: remove every OTU whose estimated occurrence probability is less than 0.8 — the 'removal of negatives' QC step that drops likely contaminant or sporadically detected sequences. Write the surviving OTU counts to filtered_otu_table.csv, preserving the original sample columns and taxonomic annotation columns, so the filtered sequence set (over 10^8 sequences) feeds taxonomic grouping and downstream community analysis. Rationale: the 80% occurrence-probability cutoff separates reproducibly detected taxa from replicate-level noise; in the Monterey Bay 2015–2016 multilocus amplicon time-series this produced the dataset of 663 taxonomic groups (Family or higher).

## Related tools

- **R** (Used to load the raw OTU count table and PCR replicate metadata, estimate per-OTU occurrence probabilities from replicate detections as binomial draws, apply the <0.8 removal threshold, and write filtered_otu_table.csv) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
write.csv(subset(otu_table, occurrence_probability >= 0.8), "filtered_otu_table.csv", row.names = FALSE)
```

## Evaluation signals

- Every retained OTU has an estimated occurrence probability >= 0.8; recomputing binomial/site-occupancy probabilities from replicate detection frequencies reproduces the keep/drop decisions.
- No OTU with estimated occurrence probability < 0.8 survives in filtered_otu_table.csv.
- Output schema check: filtered_otu_table.csv preserves the original sample columns and taxonomic annotation columns; only OTU rows are removed.
- Dropped OTUs are concentrated among those detected in few PCR replicates (putative contaminants or sporadic detections), while consistently detected taxa are retained.
- Downstream scale matches the article: the filtered sequence set (over 10^8 sequences) supports identification of 663 taxonomic groups at Family or higher.

## Limitations

- OTUs with <80% estimated occurrence probability are discarded, so genuinely rare taxa that are only sporadically detected across PCR replicates may be removed along with contaminants.
- The binomial model assumes independent replicate detections, but the article notes amplification is subject to substantial primer biases, inhibitors, and variable template abundances, which can bias occurrence-probability estimates.
- Even after filtering, detection is incomplete — 'no method—including this one—identifies all taxa in an ecosystem.'

## Evidence

- [readme] Repository scope covers quality filtering: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [abstract] Amplicon time-series context: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [other] Site-occupancy model and cutoff: [evidence span withheld — anchor status: not_found]
- [other] Removal rule for low-probability OTUs: [evidence span withheld — anchor status: not_found]
- [discussion] PCR amplification bias caveat: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
- [abstract] Dataset scale after filtering: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
