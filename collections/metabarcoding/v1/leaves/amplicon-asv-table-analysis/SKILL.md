---
name: amplicon-asv-table-analysis
description: Use when finished ASV/OTU tables from a DADA2 pipeline (e.g. dadasnake
  output) must be validated against sequencing-depth subsamples or ground-truth mock
  community compositions to detect richness or abundance artefacts.
when_to_use_negative:
- Input is raw FASTQ reads or unprocessed read files — pipeline steps (primer removal
  with cutadapt, DADA2 quality filtering, chimera removal) belong to dadasnake processing,
  not this post-hoc table analysis
- The community has no known ground-truth composition, so observed-vs-expected abundance
  correspondence cannot be quantified
- The dataset is a single sequencing depth with no subsampled depth series, so rarefaction-curve
  flattening across depths cannot be assessed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: R package providing rarefy/rarecurve to compute rarefaction curves of observed
    ASV richness against reads sampled
- name: dadasnake
  role: Source of the finished DADA2-derived ASV tables for every sequencing-depth
    subsample of the mock community datasets
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Algorithm underlying the ASV delineation and taxonomic classification whose
    output tables are benchmarked
  repo: http://benjjneb.github.io/dada2/
merged_aliases:
- amplicon-asv-table-aggregation
merged_alias_records:
- alias: amplicon-asv-table-aggregation
  slug: amplicon-asv-table-aggregation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:14c51ca86ab5accc98de82da2c0d0aff8e4565f010a030bfb8d51e6f5581aad9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - dadasnake
  - vegan
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# amplicon-asv-table-analysis

## Summary

Post-processing analysis of DADA2-derived ASV tables to benchmark whether they faithfully represent sequencing depth-richness relationships and true taxon abundances, using mock community datasets with known ground-truth composition.

## When to use

Use when finished ASV/OTU tables from a DADA2 pipeline (e.g. dadasnake output) must be validated against sequencing-depth subsamples or ground-truth mock community compositions to detect richness or abundance artefacts.

## When NOT to use

- Input is raw FASTQ reads or unprocessed read files — pipeline steps (primer removal with cutadapt, DADA2 quality filtering, chimera removal) belong to dadasnake processing, not this post-hoc table analysis
- The community has no known ground-truth composition, so observed-vs-expected abundance correspondence cannot be quantified
- The dataset is a single sequencing depth with no subsampled depth series, so rarefaction-curve flattening across depths cannot be assessed

## Inputs

- DADA2/dadasnake-produced final ASV abundance tables for sequencing-depth subsamples (e.g. ERR777696 100–1,600,000 read pairs; PRJNA626434 100–40,000 reads)
- DADA2 taxonomic classification output for aggregating ASVs to taxa
- Ground-truth mock community compositions deposited at Zenodo [60]

## Outputs

- rarefaction_curves.png — rarefaction curves of observed ASV richness across all depth subsamples per dataset
- mock_community_relative_abundances.csv — combined observed-vs-expected per-taxon relative read abundance table for both mock communities
- Quantitative plateau/richness-increase comparison per curve

## How to apply

Load the final ASV tables produced by dadasnake/DADA2 (e.g. from Zenodo [60]) for each sequencing-depth subsample of the mock communities — ERR777696 downsampled from 100 to 1,600,000 read pairs and PRJNA626434 from 100 to 40,000 reads. For the rarefaction analysis, use vegan (rarefy/rarecurve) in R to compute observed ASV richness versus reads sampled at each depth, plot all depth subsamples on shared axes, and quantify the plateau of each curve against the continued increase in true richness at greater sequencing depth. For abundance validation, aggregate ASV counts to assigned taxa using the DADA2 taxonomic classification output, normalize to relative read abundances, join with the ground-truth mock compositions from Zenodo [60], and check for deviation from expected composition (prokaryotic community) or multi-order-of-magnitude spread despite equal input (fungal community).

## Related tools

- **vegan** (R package providing rarefy/rarecurve to compute rarefaction curves of observed ASV richness against reads sampled)
- **dadasnake** (Source of the finished DADA2-derived ASV tables for every sequencing-depth subsample of the mock community datasets) — https://github.com/a-h-b/dadasnake
- **DADA2** (Algorithm underlying the ASV delineation and taxonomic classification whose output tables are benchmarked) — http://benjjneb.github.io/dada2/

## Examples

```
Rscript -e "library(vegan); asv <- read.delim('asv_table.tsv', row.names=1); rarecurve(t(asv), step=1000)"
```

## Evaluation signals

- Rarefaction curves derived from the finished ASV tables must visibly flatten with increasing read numbers while true richness continues to increase with sequencing depth
- The rarefaction figure (rarefaction_curves.png) shows curves for all depth subsamples of each dataset (ERR777696 and PRJNA626434) on the same axes
- The output CSV contains per-taxon relative read abundances for both mock communities covering all ground-truth taxa
- Fungal mock community relative abundances span several orders of magnitude despite equal input proportions
- Prokaryotic mock community relative read abundances show no correlation with the expected unequal composition

## Limitations

- Rarefaction curves and richness estimates based on finished DADA2 ASV tables flatten even though greater real sequencing depth would yield more observed ASVs, so they require caution
- Relative read abundances of ASVs can misrepresent true taxon abundances, as shown by the lack of correlation in the unequal-composition prokaryotic mock community
- Downsampling for depth series is an optional preprocessing step (via seqtk) that must be run beforehand to generate the depth-subsample inputs

## Evidence

- [supplementary] Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60].: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [methods] final_table_filtering|||||postprocessing||settings for filtering the final ASV and/or OTU tables: "final_table_filtering|||||postprocessing||settings for filtering the final ASV and/or OTU tables"
- [readme] Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts.: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
- [abstract] the delineation of exact sequence variants by using the favorably benchmarked and widely used DADA2 algorithm: "the delineation of exact sequence variants by using the favorably benchmarked and widely used DADA2 algorithm"
