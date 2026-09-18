---
name: taxonomic-read-assignment-specificity
description: Use when benchmarking 16S amplicon classification pipelines against synthetic
  mock community ground truth and you need per-pairing read assignment specificity
  plus genus- and species-level detection sensitivity metrics.
when_to_use_negative:
- Input is already a curated feature/OTU table rather than raw 16S amplicon reads
  — specificity must be computed from profiler-assigned reads against mock ground
  truth
- Evaluating shotgun metagenomic whole-genome data instead of PCR-amplified 16S rRNA
  gene reads, since the benchmark is specific to species-level assignment of 16S amplicon
  reads
- Real environmental or clinical communities with no ground-truth composition — the
  >=0.1% abundance sensitivity and read assignment specificity metrics require known
  mock community ground truth
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: PathoScope 2
  role: Read-mapping profiler benchmarked for taxon detection sensitivity; most sensitive
    at genus level when paired with RefSeq2018 (mean = 0.88, SD = 0.14); RefSeq libraries
    custom-configured via MetaScope
  repo: compbiomed.github.io/metascope-docs
- name: Kraken 2
  role: k-mer-based profiler benchmarked for sensitivity and specificity against SILVA,
    Greengenes, and Standard indices
  repo: github.com/aubreyodom/16SBenchmarking
- name: DADA2
  role: 16S amplicon ASV profiler benchmarked (QIIME 2 DADA2 plugin) as a lower-performing
    comparator
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: Naive Bayes classifier profiler benchmarked with SILVA and Greengenes indices
    obtained from the QIIME 2 website
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Naive Bayes classifier profiler benchmarked with SILVA and Greengenes reference
    files from the Mothur website
  repo: github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: Provides the download_refseq() function used to build up-to-date PathoScope
    RefSeq libraries for the benchmarking
  repo: compbiomed.github.io/metascope-docs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:d2192cf37ecfa04b20eed09df0a6f884ac63d3a93fa89d3f667ca0f8f1e69527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/compbiomed.github.io/metascope-docs
  tools:
  - DADA2
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2222222222222222
n_claims: 9
---

# taxonomic-read-assignment-specificity

## Summary

Benchmarking skill that quantifies read assignment specificity — the fraction of 16S amplicon reads assigned to taxa expected in a mock community ground truth — for each profiler/reference-library pairing, alongside taxon detection sensitivity at genus and species levels. It was used to show that whole-genome metagenomic profilers (PathoScope 2, Kraken 2) outperform 16S-specific tools.

## When to use

Use when benchmarking 16S amplicon classification pipelines against synthetic mock community ground truth and you need per-pairing read assignment specificity plus genus- and species-level detection sensitivity metrics.

## When NOT to use

- Input is already a curated feature/OTU table rather than raw 16S amplicon reads — specificity must be computed from profiler-assigned reads against mock ground truth
- Evaluating shotgun metagenomic whole-genome data instead of PCR-amplified 16S rRNA gene reads, since the benchmark is specific to species-level assignment of 16S amplicon reads
- Real environmental or clinical communities with no ground-truth composition — the >=0.1% abundance sensitivity and read assignment specificity metrics require known mock community ground truth

## Inputs

- Per-pairing taxon counts matrices from DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 classification runs
- Reference library indices: SILVA 138, Greengenes 13_8, Kraken Standard (9/19/2020), PathoScope RefSeq 2018 and RefSeq 2020
- Mock community ground-truth composition tables (136 publicly available mock samples with varied species richness and evenness)
- 16S amplicon FASTQ read files from the mock community sequencing datasets

## Outputs

- Per-sample read assignment specificity values (fraction of reads assigned to expected ground-truth taxa)
- Per-sample taxon detection sensitivity values (fraction of ground-truth taxa at >=0.1% relative abundance detected)
- Summary table of means and standard deviations per profiler-library pairing at genus and species levels (matching Figure 2A-D and Supplementary Tables S2-S5)

## How to apply

Load per-pairing taxon count matrices produced by DADA2, QIIME 2 (DADA2 plugin), Mothur, PathoScope 2, and Kraken 2 run against SILVA 138, Greengenes 13_8, Kraken Standard, and RefSeq (2018/2020) libraries, together with mock community ground-truth composition tables for the 136 mock samples. Collapse each counts matrix to genus- and species-level rows, tracing the taxonomic hierarchy back where profilers report at finer or coarser ranks (PathoScope, QIIME 2, and Mothur support tracing back the taxonomic hierarchy). For each sample and pairing compute read assignment specificity as the fraction of reads assigned to ground-truth taxa, and taxon detection sensitivity as the fraction of ground-truth taxa present at >=0.1% relative abundance that the profiler detects. Aggregate per-sample values into means and standard deviations per profiler-library pairing at genus and species levels, reproducing the paper's Figure 2A-D and Supplementary Tables S2-S5 summary tabulations. Validate against reported benchmarks, e.g., PathoScope+RefSeq2018 genus-level sensitivity mean of 0.88 (SD = 0.14), within rounding tolerance.

## Related tools

- **PathoScope 2** (Read-mapping profiler benchmarked for taxon detection sensitivity; most sensitive at genus level when paired with RefSeq2018 (mean = 0.88, SD = 0.14); RefSeq libraries custom-configured via MetaScope) — compbiomed.github.io/metascope-docs
- **Kraken 2** (k-mer-based profiler benchmarked for sensitivity and specificity against SILVA, Greengenes, and Standard indices) — github.com/aubreyodom/16SBenchmarking
- **DADA2** (16S amplicon ASV profiler benchmarked (QIIME 2 DADA2 plugin) as a lower-performing comparator) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (Naive Bayes classifier profiler benchmarked with SILVA and Greengenes indices obtained from the QIIME 2 website) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Naive Bayes classifier profiler benchmarked with SILVA and Greengenes reference files from the Mothur website) — github.com/aubreyodom/16SBenchmarking
- **MetaScope** (Provides the download_refseq() function used to build up-to-date PathoScope RefSeq libraries for the benchmarking) — compbiomed.github.io/metascope-docs

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- PathoScope+RefSeq2018 genus-level taxon detection sensitivity mean equals 0.88 (SD = 0.14) within rounding tolerance
- Summary table of per-pairing means and SDs matches Figure 2A-D and Supplementary Tables S2-S5 values within rounding tolerance
- Every sample-pairing specificity value lies in [0, 1], since it is a fraction of reads assigned to expected ground-truth taxa
- Sensitivity denominators consistently include only ground-truth taxa present at >=0.1% relative abundance
- PathoScope 2 and Kraken 2 outperform DADA2, QIIME 2, and Mothur in the aggregated sensitivity/specificity ranking, and SILVA/RefSeq/Kraken Standard outperform Greengenes

## Limitations

- Mock communities provide only a semblance of ground truth: pipetting variation when aliquoting spike-in DNA, species-specific lysis difficulty, PCR amplification bias, and reagent/environmental contamination (air, gloves, pipette tip boxes) introduce error at every experimental step
- RefSeq libraries retrieved from public repositories may contain contamination (e.g., T. flagellates scaffolds matching E. coli 16S by BLAST); older RefSeq libraries are recommended until newer versions are cleaned
- Kraken Standard library is comparable to the PathoScope RefSeq 2020 library, so cross-library comparisons must account for overlapping content and version dates (downloaded 8/20/2020; Standard listed under 9/19/2020)
- Greengenes 13_8 was the most recent Greengenes release at analysis time; Greengenes2 became available in 2022 and is not covered by these benchmarks

## Evidence

- [discussion] Mock communities provide only a semblance of ground truth: [evidence span withheld — anchor status: not_found]
- [discussion] PathoScope is more sensitive and specific at taxon detection: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [abstract] 136 mock samples with varied richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [abstract] Common 16S tools cannot produce high-quality genus/species calls: "the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls"
- [readme] Kraken 2 Greengenes index construction code: "kraken2-build --db ${DBNAME} --special greengenes --threads 1"
- [discussion] RefSeq retrieval and QC caveat: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
