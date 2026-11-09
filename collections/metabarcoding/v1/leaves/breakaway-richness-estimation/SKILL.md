---
name: breakaway-richness-estimation
description: Use when benchmarking 16S profiling pipelines or reference libraries
  (e.g. PathoScope, Kraken 2, DADA2, QIIME 2, Mothur vs RefSeq/SILVA/Greengenes) on
  rarefied count tables and you need species-level alpha-diversity accuracy against
  mock-community ground truth.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) rather than rarefied count tables — taxonomy
  profiling and rarefaction must happen first
- The goal is genus-level or differential-abundance comparison, not alpha-diversity
  accuracy benchmarking at the species level
- Samples lack mock-community ground truth, so no true reference values exist to compute
  deviations against
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: breakaway
  role: breakaway_nof1 estimator computes taxonomic richness on each rarefied count
    table; estimates are log-transformed before deviation analysis
- name: vegan
  role: computes Shannon and Simpson alpha-diversity indices on each rarefied count
    table via vegan::diversity()
- name: DADA2
  role: one of the 16S profiling tools whose rarefied species-level count tables are
    benchmarked by the diversity-deviation analysis
- name: PathoScope 2
  role: metagenomic profiler whose species-level estimates most closely matched true
    Shannon indices (RefSeq2020 deviation mean = 0.21)
- name: Kraken 2
  role: metagenomic profiler whose rarefied count tables are evaluated, e.g. Kraken
    2+Greengenes Shannon deviation mean = 1.05
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:2e13b45e7c14f5058c1402ce961e84425c7a730c0854cda9a37c5073cd5bab2b
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# breakaway-richness-estimation

## Summary

Estimate species-level taxonomic richness of 16S mock-community count tables with the breakaway package's breakaway_nof1 estimator (log-transformed), and judge profiler/reference-library pairings by the absolute deviation between estimated and true alpha diversity (Shannon, Simpson, richness).

## When to use

Use when benchmarking 16S profiling pipelines or reference libraries (e.g. PathoScope, Kraken 2, DADA2, QIIME 2, Mothur vs RefSeq/SILVA/Greengenes) on rarefied count tables and you need species-level alpha-diversity accuracy against mock-community ground truth.

## When NOT to use

- Input is raw sequencing data (FASTQ) rather than rarefied count tables — taxonomy profiling and rarefaction must happen first
- The goal is genus-level or differential-abundance comparison, not alpha-diversity accuracy benchmarking at the species level
- Samples lack mock-community ground truth, so no true reference values exist to compute deviations against

## Inputs

- Rarefied (10,000 reads) species-level count tables for each profiler/reference-library pairing
- Ground-truth count libraries of 1,000,000 virtual reads for the mock communities
- R environment with vegan and breakaway packages

## Outputs

- Shannon and Simpson alpha-diversity indices per sample per pairing (vegan::diversity)
- Log-transformed breakaway_nof1 richness estimates per sample per pairing
- Absolute deviations (|estimated - true|) per index and pairing
- figure3_diversity_deviations.csv with per-pairing and mean deviations

## How to apply

Load the rarefied (10,000 reads) count tables for every profiler/reference-library pairing plus the ground-truth libraries of 1,000,000 virtual reads. Compute Shannon and Simpson indices on each rarefied table with vegan::diversity(), and estimate taxonomic richness with the breakaway package's breakaway_nof1 estimator, applying the log transform to its output. Compute the same three indices on the 1,000,000-virtual-read ground-truth libraries to obtain true reference values. Calculate absolute deviations (|estimated - true|) per index and pairing, aggregate mean deviations (e.g. PathoScope RefSeq2020 Shannon deviation mean = 0.21; Kraken 2+Greengenes Shannon deviation mean = 1.05; DADA2 breakaway deviation mean = 1.37), and compare against reported reference values. Write per-pairing and mean deviations to figure3_diversity_deviations.csv.

## Related tools

- **breakaway** (breakaway_nof1 estimator computes taxonomic richness on each rarefied count table; estimates are log-transformed before deviation analysis)
- **vegan** (computes Shannon and Simpson alpha-diversity indices on each rarefied count table via vegan::diversity())
- **DADA2** (one of the 16S profiling tools whose rarefied species-level count tables are benchmarked by the diversity-deviation analysis)
- **PathoScope 2** (metagenomic profiler whose species-level estimates most closely matched true Shannon indices (RefSeq2020 deviation mean = 0.21))
- **Kraken 2** (metagenomic profiler whose rarefied count tables are evaluated, e.g. Kraken 2+Greengenes Shannon deviation mean = 1.05)

## Examples

```
library(vegan); library(breakaway); shannon <- vegan::diversity(rarefied_table, index="shannon"); log_rich <- log(breakaway_nof1(rarefied_table)$estimate)
```

## Evaluation signals

- Mean Shannon deviation for PathoScope+RefSeq2020 should be ~0.21 (and ~0.27 for RefSeq2018), the best of all evaluated pairings
- Kraken 2+Greengenes Shannon deviation mean should reproduce ~1.05 and DADA2 breakaway deviation mean ~1.37 as Figure 3 reference values
- All count tables must be rarefied to 10,000 reads before indices are computed; ground-truth libraries are 1,000,000 virtual reads
- breakaway_nof1 richness values must be log-transformed before computing deviations
- figure3_diversity_deviations.csv should contain both per-pairing and mean deviation values for Shannon, Simpson, and log breakaway_nof1

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors (pipetting variation, lysis difficulty, PCR amplification bias, reagent contamination) are introduced at every experimental step
- No analysis pipeline could theoretically exist to perfectly measure a mock community, so deviations are never zero
- Commonly used 16S bioinformatics tools cannot produce high-quality genus-level or species-level taxonomic calls, limiting all profilers to some degree

## Evidence

- [methods] Estimate taxonomic richness on each rarefied count table with the breakaway package's breakaway_nof1 estimator and apply the log transform.: [evidence span withheld — anchor status: not_found]
- [results] PathoScope most closely matched the true Shannon alpha diversity indices at the species level: [evidence span withheld — anchor status: not_found]
- [discussion] Mock communities provide only a semblance of a ground truth: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
- [readme] Kraken 2 Greengenes indices build command from the repository README: [evidence span withheld — anchor status: not_found]
