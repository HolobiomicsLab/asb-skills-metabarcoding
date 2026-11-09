---
name: metagenomic-read-classification-kraken2
description: Reference for applying Kraken 2 to classify 16S amplicon reads from mock
  community samples at species/genus level and comparing its accuracy against 16S-specific
  pipelines over matched reference libraries.
when_to_use_negative:
- Input data is not 16S amplicon sequencing data but general shotgun metagenomic reads,
  or the goal is OTU clustering rather than read-level taxonomic classification
- Only a 16S-native toolchain (e.g. DADA2, QIIME 2, Mothur) is in scope and no read-mapping
  comparison is intended
- Reference library choice must follow the current RefSeq database without contamination
  processing — the authors recommend using older RefSeq libraries until newer versions
  are corrected
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: Kraken 2
  role: Read-mapping classifier that assigns 16S amplicon reads to taxa against SILVA,
    Greengenes, Kraken Standard, and RefSeq libraries for benchmarking against 16S-native
    profilers
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Comparator read-mapping profiler; Kraken Standard library results are kept
    pairwise comparable with the PathoScope RefSeq 2020 library results
  repo: github.com/aubreyodom/16SBenchmarking
- name: DADA2
  role: 16S-native comparator pipeline (also via QIIME 2 DADA2 plugin) against which
    Kraken 2 species/genus-level accuracy is evaluated
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: 16S-native comparator pipeline trained with naïve Bayes classifiers on the
    PCR-amplified 16S rRNA gene region
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:3738b2f9583dd7df25cea8a16cbde700b7cf5a789522008fb35a35e96667510a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# metagenomic-read-classification-kraken2

## Summary

Benchmarking skill for taxonomically profiling 16S amplicon sequencing reads from mock communities with Kraken 2 read-mapping against SILVA, Greengenes, Kraken Standard, and RefSeq reference libraries, so results can be compared per sample against 16S-native profilers (DADA2, QIIME 2, Mothur) and PathoScope 2.

## When to use

Reference for applying Kraken 2 to classify 16S amplicon reads from mock community samples at species/genus level and comparing its accuracy against 16S-specific pipelines over matched reference libraries.

## When NOT to use

- Input data is not 16S amplicon sequencing data but general shotgun metagenomic reads, or the goal is OTU clustering rather than read-level taxonomic classification
- Only a 16S-native toolchain (e.g. DADA2, QIIME 2, Mothur) is in scope and no read-mapping comparison is intended
- Reference library choice must follow the current RefSeq database without contamination processing — the authors recommend using older RefSeq libraries until newer versions are corrected

## Inputs

- 16S amplicon sequencing FASTQ reads from mock community samples (136 samples from Lluch, Kozich, Fouhy, and Karstens datasets)
- Kraken 2 SILVA 138 index (Index Zone)
- Kraken 2 Standard library (downloaded 8/20/2020, from Index Zone 'Older Kraken 2 / Bracken Refseq indexes')
- Kraken 2 Greengenes 13_8 index built locally with kraken2-build

## Outputs

- Per-sample taxonomic calls at species/genus level
- Taxon-by-sample counts matrix (one CSV per profiler-reference-library pairing), pairwise comparable with PathoScope RefSeq 2020 results

## How to apply

Obtain Kraken 2 indices from the Index Zone: the SILVA 138 index, the Kraken Standard library (downloaded 8/20/2020, listed as 9/19/2020 under 'Older Kraken 2 / Bracken Refseq indexes' and comparable to the PathoScope RefSeq 2020 library), and build a Greengenes 13_8 index with 'kraken2-build --db ${DBNAME} --special greengenes --threads 1'. Run Kraken 2 over all 136 mock community 16S amplicon samples from the Lluch, Kozich, Fouhy, and Karstens datasets, then aggregate per-sample taxonomic calls into a taxon-by-sample counts matrix (CSV per profiler-library pairing) so the Kraken 2 results are pairwise comparable with PathoScope RefSeq 2020 and the 16S-native DADA2, QIIME 2 (DADA2 plugin), and Mothur results. Rationale: the paper found Kraken 2, a whole-genome metagenomics read-mapper, outperformed the 16S-specific tools even against identical reference databases, with the SILVA and RefSeq/Kraken 2 Standard libraries superior in accuracy to Greengenes.

## Related tools

- **Kraken 2** (Read-mapping classifier that assigns 16S amplicon reads to taxa against SILVA, Greengenes, Kraken Standard, and RefSeq libraries for benchmarking against 16S-native profilers) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Comparator read-mapping profiler; Kraken Standard library results are kept pairwise comparable with the PathoScope RefSeq 2020 library results) — github.com/aubreyodom/16SBenchmarking
- **DADA2** (16S-native comparator pipeline (also via QIIME 2 DADA2 plugin) against which Kraken 2 species/genus-level accuracy is evaluated) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (16S-native comparator pipeline trained with naïve Bayes classifiers on the PCR-amplified 16S rRNA gene region)

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Each Kraken 2 profiler-library pairing yields a complete taxon-by-sample counts matrix (rows = taxa, columns = 136 mock community samples)
- Kraken Standard and PathoScope RefSeq 2020 results remain pairwise comparable, since the libraries are documented as comparable
- Accuracy evaluation shows Kraken 2 outperforming DADA2, QIIME 2 (DADA2 plugin), and Mothur at genus and species level, even against identical reference databases
- SILVA and RefSeq/Kraken 2 Standard library results should score higher in sensitivity/specificity (lower error) than Greengenes-based results

## Limitations

- Mock communities provide only a semblance of 'ground truth': technical bias and errors are introduced at every experimental step (pipetting variation, species-specific lysis difficulty, PCR amplification bias, reagent and environmental contamination), so no pipeline can perfectly measure a mock community
- Directly retrieved public reference libraries may contain contamination (e.g. E. coli 16S rRNA sequence found in T. flagellates RefSeq scaffolds) demanding additional filtering; newer RefSeq versions should be processed to remove contamination before use
- A new Greengenes version (Greengenes2, released 2022) appeared during review, so Greengenes 13_8 results reflect the older library

## Evidence

- [methods] Kraken 2 Greengenes indices were obtained by running the following code: "Kraken 2 Greengenes indices were obtained by running the following code"
- [readme] kraken2-build command: "kraken2-build --db ${DBNAME} --special greengenes --threads 1"
- [readme] Kraken Standard library provenance: "The Kraken Standard indices was originally downloaded on 8/20/2020. This library may be downloaded from the [Index Zone](https://benlangmead.github.io/aws-indexes/k2)"
- [readme] Kraken Standard vs PathoScope RefSeq 2020 comparability: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
- [discussion] Kraken 2 outperforms 16S-native tools: "our results present strong evidence that PathoScope and Kraken 2 outperform QIIME 2, Mothur, and DADA2, even when comparing reads to identical reference databases"
- [abstract] Library accuracy ranking: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
