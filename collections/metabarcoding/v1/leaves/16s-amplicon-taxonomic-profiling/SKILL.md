---
name: 16s-amplicon-taxonomic-profiling
description: Use when benchmarking or reproducing taxonomic classification of 16S
  amplicon FASTQ data from synthetic mock communities across multiple profiler/reference-library
  combinations at genus and species level.
when_to_use_negative:
- Input is shotgun whole-genome metagenomic data rather than PCR-amplified 16S rRNA
  gene amplicon reads
- The goal is functional profiling or abundance estimation on an already-assembled
  feature table rather than taxonomic classification of raw 16S reads
- Reference libraries must be used exactly as retrieved from public repositories —
  the paper notes RefSeq versions require contamination removal before use
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: DADA2
  role: Amplicon-inference pipeline run across all mock community samples with paired
    reference libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: DADA2-plugin 16S pipeline run with SILVA and Greengenes indices
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: 16S pipeline whose naïve Bayes classifier is trained on the PCR-amplified
    16S rRNA gene region with SILVA 138 and Greengenes 13_8
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Read-mapping profiler run with custom-configured SILVA, Greengenes, and RefSeq
    2018/2020 libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: k-mer-based read classifier run with SILVA, Greengenes, and Kraken Standard
    libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: Provides the download_refseq() function for obtaining up-to-date RefSeq indices
  repo: compbiomed.github.io/metascope-docs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:b0470a9c15c9be544d2531ab8e3bf8c0bff462a50bd30093688281f13cf89030
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# 16s-amplicon-taxonomic-profiling

## Summary

Reproduce per-sample genus- and species-level taxonomic profiling of 16S amplicon mock community data by running five profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) against paired reference libraries (SILVA 138, Greengenes 13_8, Kraken Standard, RefSeq 2018/2020), yielding one taxon-by-sample counts matrix per profiler–library pairing.

## When to use

Use when benchmarking or reproducing taxonomic classification of 16S amplicon FASTQ data from synthetic mock communities across multiple profiler/reference-library combinations at genus and species level.

## When NOT to use

- Input is shotgun whole-genome metagenomic data rather than PCR-amplified 16S rRNA gene amplicon reads
- The goal is functional profiling or abundance estimation on an already-assembled feature table rather than taxonomic classification of raw 16S reads
- Reference libraries must be used exactly as retrieved from public repositories — the paper notes RefSeq versions require contamination removal before use

## Inputs

- Publicly available 16S amplicon mock community FASTQ datasets from Lluch, Kozich, Fouhy, and Karstens (136 samples with varied species richness and evenness)
- SILVA 138 reference indices for QIIME 2, Mothur, and Kraken 2 (Index Zone), plus custom PathoScope SILVA indices
- Greengenes 13_8 reference indices for QIIME 2 and Mothur, custom PathoScope Greengenes indices, and Kraken 2 Greengenes indices
- Kraken Standard library (downloaded 8/20/2020) and custom PathoScope RefSeq 2018/2020 libraries

## Outputs

- One taxon-by-sample CSV counts matrix per profiler–reference-library pairing (11 pairings), rows = taxa, columns = the 136 mock community samples
- Genus- and species-level taxonomic assignments per sample suitable for sensitivity/specificity and error-score benchmarking

## How to apply

Obtain the publicly available mock community 16S datasets (Lluch, Kozich, Fouhy, Karstens; 136 samples total) and prepare version-matched reference indices: SILVA 138 for QIIME 2, Mothur, and Kraken 2 (from the QIIME 2, Mothur, and Index Zone websites), custom-configured PathoScope SILVA/RefSeq/Greengenes indices from the authors' deposit, and Greengenes 13_8 for QIIME 2, Mothur, and Kraken 2 (built with 'kraken2-build --db ${DBNAME} --special greengenes --threads 1'); also load the Kraken Standard library (downloaded 8/20/2020, comparable to PathoScope RefSeq 2020). Execute DADA2 and the QIIME 2 DADA2-plugin pipelines, run Mothur with its naïve Bayes classifier trained on the PCR-amplified 16S rRNA gene region, and run PathoScope 2 and Kraken 2 read-mapping over all samples. Aggregate each profiler–library pairing's per-sample calls into a taxon-by-sample counts matrix saved as one CSV per pairing (11 pairings across 136 samples). The rationale, per the paper, is that metagenomic read-mappers (PathoScope 2, Kraken 2) benchmarked against standard 16S tools reveal which profiler/library pairs give the most accurate genus- and species-level calls.

## Related tools

- **DADA2** (Amplicon-inference pipeline run across all mock community samples with paired reference libraries) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (DADA2-plugin 16S pipeline run with SILVA and Greengenes indices) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (16S pipeline whose naïve Bayes classifier is trained on the PCR-amplified 16S rRNA gene region with SILVA 138 and Greengenes 13_8) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Read-mapping profiler run with custom-configured SILVA, Greengenes, and RefSeq 2018/2020 libraries) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (k-mer-based read classifier run with SILVA, Greengenes, and Kraken Standard libraries) — github.com/aubreyodom/16SBenchmarking
- **MetaScope** (Provides the download_refseq() function for obtaining up-to-date RefSeq indices) — compbiomed.github.io/metascope-docs

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Each of the 11 profiler–library pairings yields a complete taxon-by-sample counts matrix covering all 136 mock community samples
- Counts matrices are saved as one CSV per pairing with taxa as rows and mock community samples as columns
- Pairwise comparability between Kraken Standard and PathoScope RefSeq 2020 results is preserved, as documented in the paper
- Mothur and QIIME 2 classifiers are trained on the specific PCR-amplified 16S rRNA gene region rather than full-length references
- Benchmarking shows PathoScope 2 and Kraken 2 with SILVA or RefSeq/Kraken 2 Standard libraries outperforming DADA2, QIIME 2, and Mothur, and Greengenes underperforming SILVA

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no pipeline can perfectly measure a mock community
- Additional filtering and correction of contamination is required after retrieving reference libraries from public repositories; older RefSeq libraries are recommended until newer versions are decontaminated
- The most-used 16S bioinformatics tools cannot produce high-quality genus-level or species-level taxonomic calls
- Greengenes 13_8 (August 2013) was the latest release at submission; a new Greengenes2 version appeared in 2022 amidst review, limiting library-version currency

## Evidence

- [abstract] We evaluated the tools DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 in conjunction with reference libraries: "We evaluated the tools DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 in conjunction with reference libraries"
- [abstract] mock community sample count: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [readme] Kraken 2 Greengenes build command: "kraken2-build --db ${DBNAME} --special greengenes --threads 1"
- [readme] Kraken Standard comparability: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
- [discussion] classifier training rationale: "Both QIIME 2 and Mothur take advantage of naïve Bayes classifiers, which work most efficiently when trained on the specific region of the 16S rRNA gene amplified by PCR primers"
- [discussion] reference library contamination caveat: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
