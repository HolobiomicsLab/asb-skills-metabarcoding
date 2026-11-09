---
name: rarefied-count-table-diversity-analysis
description: Use when you have rarefied taxonomic count tables from multiple profiler/reference
  pairings (e.g., PathoScope, Kraken 2, DADA2 with RefSeq2020, RefSeq2018, SILVA,
  Greengenes) plus ground-truth mock community abundances and need to rank pairings
  by alpha-diversity deviation.
when_to_use_negative:
- Input is raw FASTQ or unfiltered ASV/OTU tables — count tables must already be rarefied
  to 10,000 reads before diversity indices are comparable
- The goal is genus-level or differential-abundance analysis rather than benchmarking
  species-level alpha-diversity accuracy against mock community ground truth
- Samples are environmental/clinical with no known true composition — absolute deviation
  from ground truth cannot be computed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3300
tools:
- name: vegan
  role: Computes Shannon and Simpson alpha-diversity indices on each rarefied count
    table via vegan::diversity()
- name: breakaway
  role: Provides the breakaway_nof1 richness estimator applied to rarefied count tables
    (log-transformed) for comparison against ground-truth richness
- name: DADA2
  role: One of the evaluated 16S profiling pipelines whose rarefied count tables are
    the input to the diversity deviation analysis
- name: QIIME 2
  role: Evaluated profiling tool (DADA2 plugin) whose count tables feed the deviation
    calculation
- name: Mothur
  role: Evaluated 16S profiling tool contributing rarefied count tables
- name: PathoScope 2
  role: Metagenomic profiler whose species-level estimates most closely matched true
    Shannon diversity (RefSeq2020 deviation mean = 0.21)
- name: Kraken 2
  role: Metagenomic classifier evaluated with Greengenes, SILVA, and Standard/RefSeq
    libraries in the deviation analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:d21bd0954f34fd0b30ffd00ba147dc2181bcc7669588dfe0287ece7c3c3fab0e
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
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rarefied-count-table-diversity-analysis

## Summary

Compute species-level alpha diversity (Shannon, Simpson, and log-transformed breakaway_nof1 richness) on rarefied (10,000-read) profiler count tables and compare them against 1,000,000-virtual-read ground-truth libraries to quantify how accurately each 16S pipeline/reference-library pairing estimates true diversity of mock communities.

## When to use

Use when you have rarefied taxonomic count tables from multiple profiler/reference pairings (e.g., PathoScope, Kraken 2, DADA2 with RefSeq2020, RefSeq2018, SILVA, Greengenes) plus ground-truth mock community abundances and need to rank pairings by alpha-diversity deviation.

## When NOT to use

- Input is raw FASTQ or unfiltered ASV/OTU tables — count tables must already be rarefied to 10,000 reads before diversity indices are comparable
- The goal is genus-level or differential-abundance analysis rather than benchmarking species-level alpha-diversity accuracy against mock community ground truth
- Samples are environmental/clinical with no known true composition — absolute deviation from ground truth cannot be computed

## Inputs

- Rarefied (10,000 reads) species-level count tables for each profiler/reference-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq2018, RefSeq2020, Kraken 2 Standard)
- Ground-truth libraries of 1,000,000 virtual reads from mock communities (136 publicly available samples with varied species richness and evenness)

## Outputs

- Per-pairing absolute deviation values for Shannon, Simpson, and log breakaway_nof1 indices
- figure3_diversity_deviations.csv containing per-pairing and mean deviation results

## How to apply

Load the rarefied (10,000 reads) count tables for every profiler/reference-library pairing and the 1,000,000-virtual-read ground-truth libraries. Compute Shannon and Simpson indices on each rarefied table with vegan::diversity(), and estimate taxonomic richness with breakaway's breakaway_nof1 estimator, applying the log transform. Compute the same three indices on the ground-truth libraries to obtain true reference values, then calculate absolute deviations (|estimated - true|) per index and pairing. Aggregate mean deviations per pairing; in this benchmark PathoScope best matched true Shannon diversity (RefSeq2020 deviation mean = 0.21; RefSeq2018 = 0.27) while Kraken 2+Greengenes Shannon deviation mean = 1.05 and DADA2 breakaway deviation mean = 1.37, so verify your aggregation against these reported reference values. Write per-pairing and mean deviations to figure3_diversity_deviations.csv.

## Related tools

- **vegan** (Computes Shannon and Simpson alpha-diversity indices on each rarefied count table via vegan::diversity())
- **breakaway** (Provides the breakaway_nof1 richness estimator applied to rarefied count tables (log-transformed) for comparison against ground-truth richness)
- **DADA2** (One of the evaluated 16S profiling pipelines whose rarefied count tables are the input to the diversity deviation analysis)
- **QIIME 2** (Evaluated profiling tool (DADA2 plugin) whose count tables feed the deviation calculation)
- **Mothur** (Evaluated 16S profiling tool contributing rarefied count tables)
- **PathoScope 2** (Metagenomic profiler whose species-level estimates most closely matched true Shannon diversity (RefSeq2020 deviation mean = 0.21))
- **Kraken 2** (Metagenomic classifier evaluated with Greengenes, SILVA, and Standard/RefSeq libraries in the deviation analysis)

## Examples

```
R: div <- vegan::diversity(rarefied_table, index="shannon"); rich <- log(breakaway::breakaway_nof1(rarefied_table)$estimate)
```

## Evaluation signals

- Shannon/Simpson values computed with vegan::diversity() on tables rarefied to exactly 10,000 reads
- breakaway_nof1 richness values are log-transformed before deviation calculation
- Mean Shannon deviation per pairing reproduces reported values, e.g. PathoScope RefSeq2020 = 0.21, RefSeq2018 = 0.27, Kraken 2+Greengenes = 1.05, and DADA2 breakaway deviation mean = 1.37
- Ground-truth reference indices computed from 1,000,000-virtual-read libraries, not from the 10,000-read rarefied tables
- Output CSV is named figure3_diversity_deviations.csv with per-pairing and mean deviation columns

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step
- No analysis pipeline could theoretically exist to perfectly measure a mock community
- Mock species relative abundances may be affected by subtle pipetting variations when aliquoting spike-in DNA
- Species vary in lysis difficulty, causing underrepresentation or absence in collected cDNA libraries, and amplification bias occurs during PCR despite universal primer design

## Evidence

- [methods] Load the rarefied (10,000 reads) count tables for every profiler/reference-library pairing and the ground-truth libraries of 1,000,000 virtual reads.: [evidence span withheld — anchor status: not_found]
- [methods] Estimate taxonomic richness on each rarefied count table with the breakaway package's breakaway_nof1 estimator and apply the log transform.: [evidence span withheld — anchor status: not_found]
- [results] PathoScope most closely matched the true Shannon alpha diversity indices at the species level (RefSeq2020 deviation mean = 0.21; RefSeq2018 deviation mean = 0.27): [evidence span withheld — anchor status: not_found]
- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [discussion] provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
