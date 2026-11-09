---
name: 16s-subregion-similarity-thresholding
description: Use when you have per-sub-region in-silico 16S amplicon FASTA files and
  need to test whether a given identity threshold (97/98/99%) recaptures the full-length
  sequence diversity before committing to a marker-gene region or OTU cutoff.
when_to_use_negative:
- Input is already an OTU table or abundance matrix — clustering has already been
  performed
- Analysis goal is taxonomic assignment or species/strain discrimination from full-length
  sequences rather than assessing sub-region diversity recapture
- Sequences contain chimeras you intend to filter — this skill explicitly disables
  chimera detection (no -uchime_denovo) to isolate the effect of identity threshold
  alone
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: USEARCH
  role: Runs cluster_otus at -id 0.97, 0.98, and 0.99 (with chimera detection disabled)
    to cluster each sub-region's in-silico amplicons and count resulting OTUs
- name: ruffus
  role: CGAT ruffus pipeline framework used to orchestrate the analysis pipelines
    for the article's figures
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:6ac730166b13cc86d20e1c89792941abfbecbffa7b46c11e5b8d3fde18e21907
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - USEARCH
  - ruffus
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# 16s-subregion-similarity-thresholding

## Summary

Clusters in-silico 16S amplicons from each variable sub-region (e.g., V4) with USEARCH cluster_otus at 97%, 98%, and 99% identity thresholds to quantify how many OTUs each sub-region/threshold combination yields relative to the number of distinct full-length ~1500 bp sequences in the source database.

## When to use

Use when you have per-sub-region in-silico 16S amplicon FASTA files and need to test whether a given identity threshold (97/98/99%) recaptures the full-length sequence diversity before committing to a marker-gene region or OTU cutoff.

## When NOT to use

- Input is already an OTU table or abundance matrix — clustering has already been performed
- Analysis goal is taxonomic assignment or species/strain discrimination from full-length sequences rather than assessing sub-region diversity recapture
- Sequences contain chimeras you intend to filter — this skill explicitly disables chimera detection (no -uchime_denovo) to isolate the effect of identity threshold alone

## Inputs

- In-silico amplicon FASTA files for each 16S sub-region (e.g., V4), produced by the in-silico amplification task
- Count of distinct full-length ~1500 bp sequences in the source reference database

## Outputs

- otu_counts.csv — per-region, per-threshold OTU counts (sub-region × identity threshold matrix)
- Reproduced values for Fig. 1d and Supplementary Fig. 3 of the article

## How to apply

Load the in-silico amplicon FASTA files for each 16S sub-region produced by the in-silico PCR step. For each sub-region dataset, run USEARCH cluster_otus at 97% identity (-id 0.97) with chimera detection disabled (no -uchime_denovo step), then repeat at -id 0.98 and -id 0.99, recording OTU counts per region. Compile the per-region, per-threshold counts into a single table (otu_counts.csv). Compare each sub-region's OTU count against the original number of distinct full-length sequences to confirm whether that region recaptures the full diversity; in this study, no sub-region recaptured the number of distinct sequences even at 99% identity, with V4 performing worst. Rationale: the 99% threshold was chosen because full-length 16S sequences clustered at 99% identity provided reasonable estimates of species-level relative abundance, so partial regions falling short at 99% indicate genuine loss of resolution.

## Related tools

- **USEARCH** (Runs cluster_otus at -id 0.97, 0.98, and 0.99 (with chimera detection disabled) to cluster each sub-region's in-silico amplicons and count resulting OTUs)
- **ruffus** (CGAT ruffus pipeline framework used to orchestrate the analysis pipelines for the article's figures) — https://github.com/cgat-developers/ruffus

## Examples

```
usearch -cluster_otus V4_amplicons.fasta -id 0.99 -otus V4_otus.fasta
```

## Evaluation signals

- At 99% identity, no sub-region's OTU count should equal the number of distinct full-length sequences in the source database (the negative control expectation from the study)
- The V4 sub-region should show the largest shortfall (worst performance) in OTU counts relative to distinct full-length sequences
- OTU counts must be monotonically non-decreasing as the identity threshold relaxes from 99% to 97% within each sub-region
- Tabulated per-region, per-threshold counts in otu_counts.csv must reproduce the values plotted in Fig. 1d and Supplementary Fig. 3
- Chimera detection must be confirmed disabled (no -uchime_denovo step) so counts reflect identity-threshold effects only

## Limitations

- None of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented by the full ~1500 bp gene, so this skill cannot validate a sub-region as diversity-equivalent to full-length sequencing
- The 99% identity threshold's ability to separate intragenomic vs. inter-genomic sequence variation was not determined for all bacterial species
- Repo README is minimal: it only states that pipelines are organised per figure and that Fig. 1-3 code is Python 2.7, so exact script names must be located in the repository

## Evidence

- [discussion] No sub-region recaptures full diversity: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [discussion] 99% clustering rationale: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [intro] In-silico sub-region assessment: "Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [readme] Repository code organization: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
