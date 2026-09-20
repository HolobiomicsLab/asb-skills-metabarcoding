---
name: chimera-detection-disabling-in-clustering
description: Use when reproducing OTU counts from in-silico 16S sub-region amplicons
  (e.g., V4) across multiple identity thresholds (0.97/0.98/0.99) to benchmark how
  much diversity clustering recaptures relative to distinct full-length ~1500 bp sequences.
when_to_use_negative:
- Input is real sequencing reads with plausible PCR/sequencing chimeras — disabling
  uchime_denovo there would inflate OTU counts with artifact sequences; this skill
  applies only to in-silico amplicon benchmarks
- Input is already an OTU count table or a feature/ASV table rather than per-sub-region
  amplicon FASTA files
edam_operation: http://edamontology.org/operation_0292
tools:
- name: USEARCH
  role: cluster_otus command run at -id 0.97/0.98/0.99 with chimera detection disabled
    to enumerate OTUs per 16S sub-region
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:979b272a83be31d9998f233c1a21fbfcc18b41a6dcfddd13f02138cb20ad2740
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - USEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# chimera-detection-disabling-in-clustering

## Summary

Clustering in-silico 16S amplicons with USEARCH cluster_otus at 97%, 98%, and 99% identity thresholds while deliberately leaving chimera detection (uchime_denovo) disabled, so that OTU counts reflect pure sequence-diversity collapse rather than chimera-filtering artifacts.

## When to use

Use when reproducing OTU counts from in-silico 16S sub-region amplicons (e.g., V4) across multiple identity thresholds (0.97/0.98/0.99) to benchmark how much diversity clustering recaptures relative to distinct full-length ~1500 bp sequences.

## When NOT to use

- Input is real sequencing reads with plausible PCR/sequencing chimeras — disabling uchime_denovo there would inflate OTU counts with artifact sequences; this skill applies only to in-silico amplicon benchmarks
- Input is already an OTU count table or a feature/ASV table rather than per-sub-region amplicon FASTA files

## Inputs

- In-silico amplicon FASTA files per 16S sub-region (e.g., V4) generated from a full-length 16S reference database
- Count of distinct full-length ~1500 bp sequences in the source database (denominator for recapture comparison)

## Outputs

- Per-region, per-threshold OTU counts compiled into otu_counts.csv
- Reproduction of Fig. 1d and Supplementary Fig. 3 OTU-count values

## How to apply

Load the in-silico amplicon FASTA files for each 16S sub-region (produced upstream by in-silico PCR against the source database). For each sub-region, run USEARCH cluster_otus three times at -id 0.97, -id 0.98, and -id 0.99, omitting the -uchime_denovo step entirely so chimera filtering cannot remove sequences and distort the count comparison. Record the OTU count per region per threshold and compile them into a single table (otu_counts.csv). Compare each sub-region's OTU count against the number of distinct full-length sequences in the source database; the rationale is that no short variable region, even at 99% identity, recaptures the full diversity, with V4 performing worst. Finally verify the tabulated counts reproduce the values plotted in Fig. 1d and Supplementary Fig. 3.

## Related tools

- **USEARCH** (cluster_otus command run at -id 0.97/0.98/0.99 with chimera detection disabled to enumerate OTUs per 16S sub-region)

## Examples

```
usearch -cluster_otus v4_amplicons.fasta -id 0.99 -otus v4_otus99.fasta
```

## Evaluation signals

- OTU count at 99% identity for every sub-region is strictly greater than 1 cluster but lower than the number of distinct full-length sequences in the source database (no sub-region fully recaptures diversity)
- V4 sub-region yields the largest shortfall from the distinct full-length sequence count among sub-regions tested
- otu_counts.csv contains one row per sub-region × threshold (0.97, 0.98, 0.99) combination with no missing cells
- Tabulated counts match the values plotted in Fig. 1d and Supplementary Fig. 3 of the article
- Re-running clustering with and without uchime_denovo confirms chimera filtering is the only disabled step; -id is set exactly to 0.97/0.98/0.99

## Limitations

- None of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene, so even the 99% threshold undercounts true sequence diversity
- The pipeline code is organized by figure within ruffus pipelines (Figures 1–3 in Python 2.7), so reproducing this figure requires the matching pipeline version
- The 99% identity threshold's ability to separate intragenomic vs. inter-genomic variation was not determined for all bacterial species in the study

## Evidence

- [discussion] none of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [intro] in-silico sub-regions unable to match full-gene accuracy: "Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved"
- [discussion] 99% identity clustering rationale: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [readme] repository pipeline organization: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
