---
name: deviation-from-ground-truth-quantification
description: Use when benchmarking 16S profiler/reference-library pairings on mock
  communities with known species composition, to rank methods by how closely their
  Shannon, Simpson, and breakaway_nof1 estimates match true alpha diversity.
when_to_use_negative:
- Samples lack ground-truth composition (natural communities, not mock communities),
  so |estimated - true| deviations cannot be defined.
- Count tables are raw/unrarefied; diversity comparisons require equal sequencing
  depth (the workflow rarefies to 10,000 reads).
- Goal is genus-level rather than species-level accuracy assessment; the deviation
  reference values here are species-level.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: vegan
  role: Computes Shannon and Simpson alpha-diversity indices on rarefied count tables
    via diversity()
- name: breakaway
  role: Estimates species richness with breakaway_nof1 on rarefied count tables; log-transformed
    before deviation calculation
- name: PathoScope 2
  role: Profiler whose species-level count tables gave the lowest deviation from true
    Shannon (RefSeq2020 mean = 0.21)
  repo: https://compbiomed.github.io/metascope-docs/index.html
- name: Kraken 2
  role: Profiler benchmarked with Greengenes and Standard (RefSeq) libraries; Kraken
    2+Greengenes Shannon deviation mean = 1.05
- name: DADA2
  role: Amplicon profiler benchmarked; its breakaway deviation mean = 1.37
- name: QIIME 2
  role: Amplicon profiler benchmarked with Greengenes and SILVA indices
- name: Mothur
  role: Amplicon profiler benchmarked with Greengenes and SILVA indices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:f68b0ab8c32dd0c89952ae9291b6731c2efb4cc58f94101873a88ba1cfdef504
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# deviation-from-ground-truth-quantification

## Summary

Quantifies how far each 16S profiler/reference-library pairing deviates from mock-community ground truth by computing absolute deviations of Shannon, Simpson, and log breakaway_nof1 alpha-diversity estimates (on 10,000-read rarefied tables) from true values derived from 1,000,000-virtual-read ground-truth libraries.

## When to use

Use when benchmarking 16S profiler/reference-library pairings on mock communities with known species composition, to rank methods by how closely their Shannon, Simpson, and breakaway_nof1 estimates match true alpha diversity.

## When NOT to use

- Samples lack ground-truth composition (natural communities, not mock communities), so |estimated - true| deviations cannot be defined.
- Count tables are raw/unrarefied; diversity comparisons require equal sequencing depth (the workflow rarefies to 10,000 reads).
- Goal is genus-level rather than species-level accuracy assessment; the deviation reference values here are species-level.

## Inputs

- Rarefied (10,000 reads) species-level count tables for each profiler/reference-library pairing
- Ground-truth count libraries of 1,000,000 virtual reads from synthetic mock communities (136 samples with varied species richness and evenness)

## Outputs

- Per-sample absolute deviations (|estimated - true|) for Shannon, Simpson, and log breakaway_nof1 per tool/library pairing
- Mean deviation per pairing and index
- figure3_diversity_deviations.csv

## How to apply

Load rarefied (10,000 reads) count tables for every profiler/reference-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 against Greengenes 13_8, SILVA 138, RefSeq 2018/2020, Kraken 2 Standard) plus 1,000,000-virtual-read ground-truth libraries. Compute Shannon and Simpson indices with vegan::diversity() and estimate richness with breakaway::breakaway_nof1 (log-transformed) on every table. Compute the same three indices on the ground-truth libraries to obtain true reference values, then take absolute deviations |estimated - true| per sample, index, and pairing. Aggregate mean deviations per pairing and check them against published reference values, e.g. PathoScope RefSeq2020 Shannon deviation mean = 0.21, Kraken 2+Greengenes Shannon deviation mean = 1.05, DADA2 breakaway deviation mean = 1.37. Write per-pairing and mean deviations to figure3_diversity_deviations.csv.

## Related tools

- **vegan** (Computes Shannon and Simpson alpha-diversity indices on rarefied count tables via diversity())
- **breakaway** (Estimates species richness with breakaway_nof1 on rarefied count tables; log-transformed before deviation calculation)
- **PathoScope 2** (Profiler whose species-level count tables gave the lowest deviation from true Shannon (RefSeq2020 mean = 0.21)) — https://compbiomed.github.io/metascope-docs/index.html
- **Kraken 2** (Profiler benchmarked with Greengenes and Standard (RefSeq) libraries; Kraken 2+Greengenes Shannon deviation mean = 1.05)
- **DADA2** (Amplicon profiler benchmarked; its breakaway deviation mean = 1.37)
- **QIIME 2** (Amplicon profiler benchmarked with Greengenes and SILVA indices)
- **Mothur** (Amplicon profiler benchmarked with Greengenes and SILVA indices)

## Examples

```
vegan::diversity(rarefied_counts, index = "shannon")
```

## Evaluation signals

- PathoScope RefSeq2020 Shannon deviation mean equals 0.21 and RefSeq2018 equals 0.27 when computed from 10,000-read rarefied species tables.
- Kraken 2 + Greengenes Shannon deviation mean equals 1.05, matching the reported Figure 3 reference value.
- DADA2 log breakaway_nof1 deviation mean equals 1.37.
- All deviation values are non-negative absolute differences against 1,000,000-virtual-read ground-truth libraries.
- Output file figure3_diversity_deviations.csv contains one mean-deviation row per profiler/reference-library pairing for each of the three indices.

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no pipeline can perfectly match the true indices.
- Mock species relative abundances may shift due to pipetting variation, differential lysis difficulty, and PCR amplification bias despite universal primers.
- Reagent and environmental contamination (air, gloves, pipette tip boxes) can complicate deviation estimates.

## Evidence

- [results] PathoScope most closely matched the true Shannon alpha diversity indices at the species level (RefSeq2020 deviation mean = 0.21; RefSeq2018 deviation mean = 0.27): [evidence span withheld — anchor status: not_found]
- [abstract] 136-sample mock community dataset: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [readme] Kraken 2 Greengenes index construction from README: "Kraken 2 Greengenes indices were obtained by running the following code:
kraken2-build --db ${DBNAME} --special greengenes --threads 1"
- [discussion] Ground truth is approximate: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
- [readme] Kraken Standard library comparable to PathoScope RefSeq 2020: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
