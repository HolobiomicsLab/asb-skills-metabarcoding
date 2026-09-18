---
name: microbial-taxonomy-abundance-analysis
description: Use when you have final ASV abundance tables from a DADA2/dadasnake amplicon
  run (e.g. mock communities ERR777696 or PRJNA626434) and a known expected taxon
  composition, and need to test whether relative read abundances reflect true taxon
  proportions.
when_to_use_negative:
- Input is still raw FASTQ reads — primer removal, quality filtering and trimming,
  and chimera removal must first run through dadasnake's DADA2-based steps before
  this skill applies
- The community has no known expected (ground-truth) composition, so no observed-vs-expected
  comparison can be quantified
- The goal is functional prediction (e.g. PICRUSt2 pathway inference) rather than
  taxonomic abundance assessment
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: Snakemake pipeline that produced the final taxonomically assigned ASV abundance
    tables and deposited the mock-community processing results and ground-truth compositions
    on Zenodo
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Algorithm underlying ASV delineation, error estimation and denoising; its
    taxonomic classification output provides the taxa to which ASV read counts are
    aggregated
  repo: http://benjjneb.github.io/dada2/
- name: Snakemake
  role: Workflow engine of dadasnake that wraps the read preprocessing through to
    taxonomically assigned OTU tables feeding this analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:fb7aa8a25ea2ccf21da39a724c19525e76eddbeb4c55371085f74253727dad44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - Snakemake
  - dadasnake
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# microbial-taxonomy-abundance-analysis

## Summary

Aggregates dadasnake/DADA2 ASV read counts to taxonomically assigned taxa, normalizes them to relative read abundances, and compares them against known ground-truth mock community compositions to quantify how faithfully read abundances reflect true taxon abundances.

## When to use

Use when you have final ASV abundance tables from a DADA2/dadasnake amplicon run (e.g. mock communities ERR777696 or PRJNA626434) and a known expected taxon composition, and need to test whether relative read abundances reflect true taxon proportions.

## When NOT to use

- Input is still raw FASTQ reads — primer removal, quality filtering and trimming, and chimera removal must first run through dadasnake's DADA2-based steps before this skill applies
- The community has no known expected (ground-truth) composition, so no observed-vs-expected comparison can be quantified
- The goal is functional prediction (e.g. PICRUSt2 pathway inference) rather than taxonomic abundance assessment

## Inputs

- Final ASV abundance tables produced by dadasnake/DADA2 (text/R data set outputs, e.g. biom, phyloseq objects, or text files) for prokaryotic run ERR777696 and fungal run PRJNA626434
- DADA2 taxonomic classification output assigning ASVs to taxa
- Ground-truth mock community composition tables from Zenodo [60] (doi=10.5281/zenodo.3986799 / 10.5281/zenodo.4190897)

## Outputs

- mock_community_relative_abundances.csv: per-taxon table of observed relative read abundances vs. expected ground-truth abundances for both the prokaryotic and fungal mock communities

## How to apply

Load the dadasnake/DADA2-produced final ASV abundance tables (prokaryotic run ERR777696 and fungal run PRJNA626434) from the deposited processing results at Zenodo [60], together with the DADA2 taxonomic classification output. Aggregate ASV read counts to their assigned taxa and normalize per-taxon counts to relative read abundances within each mock community. Join these observed abundances to the ground-truth mock community compositions from Zenodo [60] by taxon. Quantify correspondence: for the unequal-composition prokaryotic community, check whether observed relative abundances correlate with expected composition (the finding is that they do not, i.e. read abundances misrepresent true taxon abundances); for the equal-input fungal community, check whether relative abundances span several orders of magnitude despite equal input proportions. Write the combined observed-vs-expected per-taxon table to mock_community_relative_abundances.csv.

## Related tools

- **dadasnake** (Snakemake pipeline that produced the final taxonomically assigned ASV abundance tables and deposited the mock-community processing results and ground-truth compositions on Zenodo) — https://github.com/a-h-b/dadasnake
- **DADA2** (Algorithm underlying ASV delineation, error estimation and denoising; its taxonomic classification output provides the taxa to which ASV read counts are aggregated) — http://benjjneb.github.io/dada2/
- **Snakemake** (Workflow engine of dadasnake that wraps the read preprocessing through to taxonomically assigned OTU tables feeding this analysis)

## Examples

```
R -e 'ab <- read.delim("asv_table.tsv"); agg <- aggregate(reads ~ taxon, ab, sum); agg$rel <- agg$reads/sum(agg$reads); write.csv(agg, "mock_community_relative_abundances.csv", row.names=FALSE)'
```

## Evaluation signals

- Output table mock_community_relative_abundances.csv contains per-taxon relative read abundances for both mock communities covering all ground-truth taxa
- Fungal community (PRJNA626434) relative abundances span several orders of magnitude despite equal input proportions of all taxa
- Prokaryotic community (ERR777696, unequal known composition) shows no correlation between relative read abundance and expected taxon abundance
- ASV-level counts aggregate losslessly to assigned taxa (per-taxon read sums match the final ASV table totals)
- Each taxon row joins correctly to the ground-truth composition by taxon name, with no unmatched expected taxa

## Limitations

- Relative read abundances in amplicon data misrepresent true taxon abundances — the prokaryotic mock community showed no correlation with the unequal expected composition
- Even with equal input proportions, fungal relative read abundances varied over several orders of magnitude, so equal composition cannot be assumed from equal counts
- Results depend on dadasnake/DADA2 defaults (quality filtering, chimera removal, final_table_filtering settings) used to build the ASV tables

## Evidence

- [supplementary] Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60].: "Processing results of the mock community datasets, the ground-truth mock community compositions, and the scripts to visualize the use case datasets are available from Zenodo [60]."
- [abstract] The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi.: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
- [readme] Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts.: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
