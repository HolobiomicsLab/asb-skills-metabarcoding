---
name: alpha-diversity-index-computation
description: Use when rarefied 16S amplicon count tables from mock-community profilers
  (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) need Shannon/Simpson/richness deviation
  scores against a ground-truth library.
when_to_use_negative:
- Input is raw 16S FASTQ rather than an already-profiled, rarefied count table — taxonomic
  profiling (PathoScope/Kraken 2/DADA2) must run first
- The goal is genus/species-level classification accuracy (sensitivity, specificity,
  error score) rather than alpha-diversity deviation analysis
- Beta-diversity or differential-abundance questions, which are outside this deviation-from-ground-truth
  alpha-diversity workflow
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: vegan
  role: Computes Shannon and Simpson alpha-diversity indices on each rarefied count
    table via diversity()
- name: breakaway
  role: Provides the breakaway_nof1 taxonomic richness estimator applied (log-transformed)
    to each rarefied count table
- name: PathoScope 2
  role: Profiler whose RefSeq2020/RefSeq2018 pairings produced the lowest Shannon
    deviation means at the species level
- name: Kraken 2
  role: Profiler whose reference-library pairings (e.g. Greengenes) are benchmarked
    via their Shannon deviation means
- name: DADA2
  role: Profiler whose pairing yields the reference breakaway deviation mean (1.37)
    used to verify the workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:e56260a3f93a16fa386fd16cf613dc47d61cbd6c4a65982a4bf2c0a7c968b584
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  - breakaway
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# alpha-diversity-index-computation

## Summary

Computes species-level alpha diversity (Shannon, Simpson via vegan::diversity; richness via breakaway_nof1 with log transform) on rarefied 10,000-read count tables from 16S mock-community profilers and on 1,000,000-virtual-read ground-truth libraries, then quantifies per-pairing absolute deviations to rank profiler/reference-library accuracy.

## When to use

Use when rarefied 16S amplicon count tables from mock-community profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) need Shannon/Simpson/richness deviation scores against a ground-truth library.

## When NOT to use

- Input is raw 16S FASTQ rather than an already-profiled, rarefied count table — taxonomic profiling (PathoScope/Kraken 2/DADA2) must run first
- The goal is genus/species-level classification accuracy (sensitivity, specificity, error score) rather than alpha-diversity deviation analysis
- Beta-diversity or differential-abundance questions, which are outside this deviation-from-ground-truth alpha-diversity workflow

## Inputs

- Rarefied (10,000 reads) taxonomic count tables, one per profiler/reference-library pairing (PathoScope 2, Kraken 2, DADA2, QIIME 2, Mothur x Greengenes 13_8, SILVA 138, RefSeq 2018/2020, Kraken 2 Standard)
- Ground-truth libraries of 1,000,000 virtual reads per mock community
- Publicly available 16S mock community data (136 samples with varied species richness and evenness)

## Outputs

- Per-pairing absolute deviations (|estimated - true|) for Shannon, Simpson, and log breakaway_nof1
- Mean deviation per profiler/reference-library pairing per index
- figure3_diversity_deviations.csv

## How to apply

Load the rarefied (10,000 reads) count tables for every profiler/reference-library pairing (e.g. PathoScope+RefSeq2020, Kraken 2+Greengenes) plus the 1,000,000-virtual-read ground-truth libraries. Compute Shannon and Simpson indices on each rarefied table with vegan::diversity(), and estimate species richness with breakaway::breakaway_nof1() followed by a log transform. Apply the same three indices to the ground-truth libraries to obtain true reference values, then compute absolute deviations |estimated - true| per index and per pairing. Aggregate mean deviations per pairing and sanity-check against Figure 3 reference values (PathoScope RefSeq2020 Shannon deviation mean = 0.21; Kraken 2+Greengenes = 1.05; DADA2 breakaway = 1.37). Write per-pairing and mean deviations to figure3_diversity_deviations.csv.

## Related tools

- **vegan** (Computes Shannon and Simpson alpha-diversity indices on each rarefied count table via diversity())
- **breakaway** (Provides the breakaway_nof1 taxonomic richness estimator applied (log-transformed) to each rarefied count table)
- **PathoScope 2** (Profiler whose RefSeq2020/RefSeq2018 pairings produced the lowest Shannon deviation means at the species level)
- **Kraken 2** (Profiler whose reference-library pairings (e.g. Greengenes) are benchmarked via their Shannon deviation means)
- **DADA2** (Profiler whose pairing yields the reference breakaway deviation mean (1.37) used to verify the workflow)

## Examples

```
shannon_est <- vegan::diversity(rarefied_counts, index = "shannon"); richness <- log(breakaway::breakaway_nof1(rarefied_counts)$estimate)
```

## Evaluation signals

- PathoScope + RefSeq2020 Shannon deviation mean reproduces the reported value of 0.21 (RefSeq2018 = 0.27), lowest among all pairings
- Kraken 2 + Greengenes Shannon deviation mean reproduces Figure 3 reference value of 1.05
- DADA2 breakaway deviation mean reproduces Figure 3 reference value of 1.37
- Ground-truth indices are computed on 1,000,000-virtual-read libraries while estimated indices use 10,000-read rarefied tables — verify rarefaction depth and library sizes before deviation computation
- figure3_diversity_deviations.csv contains per-pairing rows and mean-deviation aggregates for all three indices

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias (pipetting variation in spike-in aliquoting, lysis difficulty, PCR amplification bias) and contamination are introduced at every experimental step, so no pipeline can perfectly measure a mock community
- Diversity estimates depend on the quality of the upstream reference library; newer RefSeq versions may contain contamination and should be processed before use
- The 10,000-read rarefaction depth limits richness estimation relative to the 1,000,000-read ground-truth libraries

## Evidence

- [results] PathoScope most closely matched the true Shannon alpha diversity indices at the species level: [evidence span withheld — anchor status: not_found]
- [abstract] the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls: "the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls"
- [discussion] PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
