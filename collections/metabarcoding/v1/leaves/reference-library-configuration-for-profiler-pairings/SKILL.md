---
name: reference-library-configuration-for-profiler-pairings
description: Use when benchmarking taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope
  2, Kraken 2) on 16S amplicon mock community data and you need the specific reference
  indices each profiler–library pairing requires.
when_to_use_negative:
- Input is already a taxonomic feature table or counts matrix — reference library
  configuration is upstream of profiling
- Your profiling targets shotgun metagenomic samples other than 16S mock communities;
  library preparation and classifier training choices (e.g. region-specific naïve
  Bayes training) may not transfer
- You need the newest Greengenes2 or SILVA 138.1 releases — the paper used Greengenes
  13_8 (August 2013) and pinned SILVA 138 indices, and Greengenes2 was released only
  amidst review
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: DADA2
  role: ASV-based 16S profiler run with paired SILVA/Greengenes reference libraries
    across all mock community samples
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: DADA2-plugin pipeline run with SILVA and Greengenes indices from the QIIME
    2 website
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Profiler whose naïve Bayes classifier is trained on the PCR-amplified 16S
    rRNA gene region using Mothur-website SILVA/Greengenes indices
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Read-mapping profiler run with custom-configured SILVA, Greengenes, RefSeq
    2018, and RefSeq 2020 libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: k-mer read-mapping profiler run with Index Zone SILVA, custom-built Greengenes,
    and Kraken Standard libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: Provides the download_refseq() function to obtain up-to-date RefSeq indices
    for PathoScope
  repo: compbiomed.github.io/metascope-docs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:cd5ad81de19e1065edaa2e6ab28b4b799b96465bbb4464ec3c583db9201486fb
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
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# reference-library-configuration-for-profiler-pairings

## Summary

Acquire and configure the correct version-matched reference library (SILVA 138, Greengenes 13_8, Kraken Standard, RefSeq 2018/2020) for each profiler in a 16S taxonomic-profiling benchmark, so that every pipeline–library pairing yields a comparable taxon-by-sample counts matrix.

## When to use

Use when benchmarking taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) on 16S amplicon mock community data and you need the specific reference indices each profiler–library pairing requires.

## When NOT to use

- Input is already a taxonomic feature table or counts matrix — reference library configuration is upstream of profiling
- Your profiling targets shotgun metagenomic samples other than 16S mock communities; library preparation and classifier training choices (e.g. region-specific naïve Bayes training) may not transfer
- You need the newest Greengenes2 or SILVA 138.1 releases — the paper used Greengenes 13_8 (August 2013) and pinned SILVA 138 indices, and Greengenes2 was released only amidst review

## Inputs

- Publicly available mock community 16S amplicon datasets (Lluch, Kozich, Fouhy, Karstens; 136 samples total)
- SILVA 138 reference indices formatted for QIIME 2, Mothur, and Kraken 2
- Custom-configured PathoScope SILVA indices
- Greengenes 13_8 reference indices for QIIME 2 and Mothur
- Custom-configured PathoScope Greengenes and RefSeq 2018/2020 indices
- Kraken Standard library downloaded 8/20/2020 from the Index Zone

## Outputs

- Eleven taxon-by-sample counts matrices (one CSV per pipeline–library pairing) with rows as taxa and columns as the 136 mock community samples
- Configured, version-pinned reference libraries ready for each profiler

## How to apply

Obtain SILVA 138 indices from each tool's canonical source: QIIME 2 indices from the QIIME 2 website (2022.2 data resources), Mothur indices from the Mothur website, Kraken 2 SILVA indices from the Index Zone, and the custom-configured PathoScope SILVA indices from the authors' deposit. For Greengenes 13_8, get QIIME 2 and Mothur indices from their respective websites, build the Kraken 2 index with 'kraken2-build --db ${DBNAME} --special greengenes --threads 1', and use the custom PathoScope Greengenes indices. Load the Kraken Standard library downloaded 8/20/2020 from the Index Zone 'Older Kraken 2 / Bracken Refseq indexes' section (comparable to the PathoScope RefSeq 2020 library), plus the custom PathoScope RefSeq 2018 and 2020 libraries; up-to-date RefSeq indices can be regenerated with the download_refseq() function from the MetaScope package. Train Mothur's naïve Bayes classifier on the specific PCR-amplified 16S rRNA gene region, then run each of the 11 pipeline–library pairings over the 136 mock community samples and aggregate per-pairing taxonomic calls into a taxon-by-sample counts CSV.

## Related tools

- **DADA2** (ASV-based 16S profiler run with paired SILVA/Greengenes reference libraries across all mock community samples) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (DADA2-plugin pipeline run with SILVA and Greengenes indices from the QIIME 2 website) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Profiler whose naïve Bayes classifier is trained on the PCR-amplified 16S rRNA gene region using Mothur-website SILVA/Greengenes indices) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Read-mapping profiler run with custom-configured SILVA, Greengenes, RefSeq 2018, and RefSeq 2020 libraries) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (k-mer read-mapping profiler run with Index Zone SILVA, custom-built Greengenes, and Kraken Standard libraries) — github.com/aubreyodom/16SBenchmarking
- **MetaScope** (Provides the download_refseq() function to obtain up-to-date RefSeq indices for PathoScope) — compbiomed.github.io/metascope-docs

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Each of the 11 pipeline–library pairings yields a complete counts matrix with taxa as rows and all 136 mock community samples as columns
- The Kraken Standard library (downloaded 8/20/2020) and the PathoScope RefSeq 2020 library remain pairwise comparable, as documented in the paper
- Mothur classifier was trained on the specific PCR-amplified 16S rRNA gene region rather than full-length references
- Library versions match the paper's pins: SILVA 138, Greengenes 13_8 (August 2013), RefSeq 2018 and 2020 PathoScope libraries
- Every result is saved as one CSV per profiler–library pairing so accuracy (sensitivity/specificity at genus and species level) can be scored against mock community composition

## Limitations

- The Kraken Standard index is listed on the Index Zone under date 9/19/2020 even though it was downloaded 8/20/2020 — version bookkeeping is confusing
- PathoScope SILVA, Greengenes, and RefSeq indices are custom-configured and must be retrieved from the authors' deposits, not public tool websites
- Directly retrieved public libraries may require additional filtering and correction; the authors found RefSeq contamination (e.g. E. coli 16S rRNA gene sequence in T. flagellates scaffolds) and recommend using older RefSeq libraries until newer versions are cleaned
- Mock communities provide only a semblance of ground truth: pipetting variation, lysis bias, PCR amplification bias, and reagent contamination affect expected abundances

## Evidence

- [other] Acquire and prepare the SILVA 138 reference indices for QIIME 2 (QIIME 2 website), Mothur (Mothur website), and Kraken 2 (Index Zone): [evidence span withheld — anchor status: not_found]
- [readme] kraken2-build --db ${DBNAME} --special greengenes --threads 1: "Kraken 2 Greengenes indices were obtained by running the following code"
- [readme] The Kraken Standard indices was originally downloaded on 8/20/2020.: "The Kraken Standard indices was originally downloaded on 8/20/2020. This library may be downloaded from the [Index Zone]"
- [discussion] Both QIIME 2 and Mothur take advantage of naïve Bayes classifiers, which work most efficiently when trained on the specific region of the 16S rRNA gene amplified by PCR primers: "Both QIIME 2 and Mothur take advantage of naïve Bayes classifiers, which work most efficiently when trained on the specific region of the 16S rRNA gene amplified by PCR primers"
- [other] The 136 mock community samples were analyzed using 11 distinct pairings of analysis pipelines and reference libraries: [evidence span withheld — anchor status: not_found]
