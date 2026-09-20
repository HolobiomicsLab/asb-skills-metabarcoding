---
name: figure-reproducibility-of-otu-counts
description: Use when in-silico 16S amplicon FASTA files exist for one or more variable-region
  sub-regions and you need to quantify how clustering identity threshold affects OTU
  counts relative to full-length 16S diversity.
when_to_use_negative:
- Input is real sequencing reads or a feature/OTU abundance table rather than in-silico
  amplicon FASTA files from a reference database
- You need chimera-filtered OTU counts, since this protocol explicitly disables the
  -uchime_denovo step to match the paper's in-silico comparison
- The goal is taxonomic assignment accuracy rather than OTU-count/diversity recapture
  across identity thresholds
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
tools:
- name: USEARCH
  role: cluster_otus is run per sub-region at -id 0.97/0.98/0.99 with chimera detection
    disabled to generate OTU counts
- name: ruffus (CGAT)
  role: pipeline framework organising the figure-level analysis tasks in the paper's
    repository
  repo: https://github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: paper's repository containing the ruffus pipelines underlying Figures 1-3
    (Python 2.7)
  repo: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:60588fb3b4bb130d760c7de1c2d119cf9b517117d4fd787f4cc4043b17443de4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - USEARCH
  - ruffus (CGAT)
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# figure-reproducibility-of-otu-counts

## Summary

Reproduces the paper's in-silico OTU-counting analysis by clustering 16S sub-region amplicons with USEARCH cluster_otus at 97%, 98%, and 99% identity and comparing per-region OTU counts against the number of distinct full-length sequences, as plotted in Fig. 1d and Supplementary Fig. 3.

## When to use

Use when in-silico 16S amplicon FASTA files exist for one or more variable-region sub-regions and you need to quantify how clustering identity threshold affects OTU counts relative to full-length 16S diversity.

## When NOT to use

- Input is real sequencing reads or a feature/OTU abundance table rather than in-silico amplicon FASTA files from a reference database
- You need chimera-filtered OTU counts, since this protocol explicitly disables the -uchime_denovo step to match the paper's in-silico comparison
- The goal is taxonomic assignment accuracy rather than OTU-count/diversity recapture across identity thresholds

## Inputs

- In-silico amplicon FASTA files for each 16S sub-region (from task_001)
- Source database of distinct full-length ~1500 bp 16S sequences and its distinct-sequence count

## Outputs

- otu_counts.csv: per-region, per-threshold (97%, 98%, 99%) OTU counts
- Comparison confirming no sub-region recaptures the full-length distinct-sequence count

## How to apply

Load the in-silico amplicon FASTA files for each 16S sub-region produced by the in-silico primer-trimming task. For each sub-region, run USEARCH cluster_otus at -id 0.97, 0.98, and 0.99, explicitly disabling de novo chimera detection (no -uchime_denovo step) to match the article's protocol, and record the resulting OTU count per region and threshold. Compile the per-region, per-threshold counts into a single table (otu_counts.csv). Compare each sub-region's OTU count against the original number of distinct full-length ~1500 bp sequences in the source database; the article's finding is that no sub-region, even at 99% identity, recaptures full-length diversity, with V4 performing worst. Verify the tabulated counts reproduce the values plotted in Fig. 1d and Supplementary Fig. 3. Pipelines in the paper's repository (Figures 1-3) are Python 2.7 ruffus pipelines built on the CGAT ruffus framework.

## Related tools

- **USEARCH** (cluster_otus is run per sub-region at -id 0.97/0.98/0.99 with chimera detection disabled to generate OTU counts)
- **ruffus (CGAT)** (pipeline framework organising the figure-level analysis tasks in the paper's repository) — https://github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (paper's repository containing the ruffus pipelines underlying Figures 1-3 (Python 2.7)) — https://github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Examples

```
usearch -cluster_otus amplicons_V4.fasta -id 0.99 -otus V4_otus.fasta -relabel OTU_ && grep -c '>' V4_otus.fasta >> otu_counts.csv
```

## Evaluation signals

- Number of OTUs is recorded for every (sub-region, identity-threshold) pair at 0.97, 0.98, and 0.99
- At 99% identity, every sub-region's OTU count still exceeds the number of distinct full-length sequences in the source database (none recaptures full diversity), with V4 performing worst
- Tabulated counts in otu_counts.csv reproduce the values plotted in Fig. 1d and Supplementary Fig. 3
- No -uchime_denovo chimera-filtering step was applied, matching the in-silico protocol
- Chimera detection is consistently disabled across all regions and thresholds so counts are directly comparable

## Limitations

- No sub-region covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene, even at 99% identity
- The 99% identity threshold's ability to separate intragenomic vs. inter-genomic variation was not determined for all bacterial species
- Pipelines underlying Figures 1-3 are written in Python 2.7 and run on the CGAT ruffus framework, which may require environment setup

## Evidence

- [discussion] none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [intro] Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the: "commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [readme] Code is organised into a set of ruffus pipelines pertaining to different figures. Each pipeline is designed to be run using a CGAT ruffus framework. The code underlying Figures 1-3 is written in Python 2.7.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
