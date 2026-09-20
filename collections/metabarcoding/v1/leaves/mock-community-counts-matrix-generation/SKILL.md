---
name: mock-community-counts-matrix-generation
description: Use when benchmarking 16S amplicon taxonomic profilers on synthetic mock
  communities and you need comparable per-sample taxon abundance counts matrices per
  profiler–reference-library pairing as the substrate for genus- and species-level
  accuracy evaluation.
when_to_use_negative:
- Input data are not synthetic mock communities with known composition — the skill's
  counts matrices feed accuracy/sensitivity/specificity evaluation that requires a
  reference 'ground truth'
- A feature/counts table already exists and only downstream genus- or species-level
  accuracy scoring is needed
- The analysis targets shotgun metagenomic (whole-genome) data rather than PCR-amplified
  16S rRNA gene amplicons
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: DADA2
  role: Amplicon denoising/ASV profiler run with its paired reference library across
    all mock community samples
- name: QIIME 2
  role: DADA2-plugin pipeline run against SILVA and Greengenes indices; uses a naïve
    Bayes classifier trained on the PCR-amplified 16S region
- name: Mothur
  role: Profiler run with SILVA 138 and Greengenes 13_8, classifier trained on the
    PCR-amplified 16S rRNA gene region
- name: PathoScope 2
  role: Read-mapping profiler run with custom-configured SILVA, Greengenes, RefSeq
    2018, and RefSeq 2020 libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: k-mer read classifier run with SILVA, locally built Greengenes, Kraken Standard,
    and RefSeq-comparable libraries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:8d37f05c47c4019cc5ab96f8e50ec7f527ce844699504ce9e4b039aa765ddec6
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
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mock-community-counts-matrix-generation

## Summary

Reproduces per-sample taxon-by-sample counts matrices for the 11 pipeline/reference-library pairings (DADA2, QIIME 2 DADA2-plugin, Mothur, PathoScope 2, Kraken 2 crossed with SILVA 138, Greengenes 13_8, Kraken Standard, and RefSeq 2018/2020) applied to the 136 publicly available mock community 16S amplicon samples from Lluch, Kozich, Fouhy, and Karstens, as benchmarked by Odom et al. 2023.

## When to use

Use when benchmarking 16S amplicon taxonomic profilers on synthetic mock communities and you need comparable per-sample taxon abundance counts matrices per profiler–reference-library pairing as the substrate for genus- and species-level accuracy evaluation.

## When NOT to use

- Input data are not synthetic mock communities with known composition — the skill's counts matrices feed accuracy/sensitivity/specificity evaluation that requires a reference 'ground truth'
- A feature/counts table already exists and only downstream genus- or species-level accuracy scoring is needed
- The analysis targets shotgun metagenomic (whole-genome) data rather than PCR-amplified 16S rRNA gene amplicons

## Inputs

- 136 publicly available mock community 16S amplicon sequencing samples (Lluch, Kozich, Fouhy, Karstens datasets) with varied species richness and evenness
- SILVA 138 reference indices for QIIME 2, Mothur, and Kraken 2 (Index Zone), and custom PathoScope SILVA indices
- Greengenes 13_8 indices for QIIME 2 and Mothur; custom PathoScope Greengenes indices; Kraken 2 Greengenes index built locally
- Kraken Standard library (downloaded 8/20/2020) and custom PathoScope RefSeq 2018 and RefSeq 2020 libraries

## Outputs

- One taxon-by-sample counts matrix CSV per pipeline/reference-library pairing (11 pairings total), rows = taxa, columns = the 136 mock community samples
- Pairwise-comparable Kraken Standard vs. PathoScope RefSeq 2020 results preserved for cross-tool comparison

## How to apply

Obtain the 136 mock community 16S samples from the Lluch, Kozich, Fouhy, and Karstens public datasets, then acquire each reference library exactly as documented: SILVA 138 indices from the QIIME 2, Mothur, and Kraken 2 (Index Zone) websites plus custom PathoScope SILVA indices; Greengenes 13_8 from QIIME 2 and Mothur sites, built for Kraken 2 with 'kraken2-build --db ${DBNAME} --special greengenes --threads 1'; Kraken Standard downloaded 8/20/2020; and custom PathoScope RefSeq 2018/2020 libraries. Run each pairing over all samples: DADA2 and the QIIME 2 DADA2-plugin with their paired libraries; Mothur with SILVA 138 and Greengenes 13_8, training its naïve Bayes classifier on the specific PCR-amplified 16S rRNA gene region; and PathoScope 2 and Kraken 2 read-mapping against the SILVA, Greengenes, Kraken Standard, and RefSeq libraries. Aggregate each pairing's per-sample taxonomic calls into a taxon-by-sample matrix and save one CSV per pairing, keeping Kraken Standard and PathoScope RefSeq 2020 comparable since they are built from equivalent RefSeq content.

## Related tools

- **DADA2** (Amplicon denoising/ASV profiler run with its paired reference library across all mock community samples)
- **QIIME 2** (DADA2-plugin pipeline run against SILVA and Greengenes indices; uses a naïve Bayes classifier trained on the PCR-amplified 16S region)
- **Mothur** (Profiler run with SILVA 138 and Greengenes 13_8, classifier trained on the PCR-amplified 16S rRNA gene region)
- **PathoScope 2** (Read-mapping profiler run with custom-configured SILVA, Greengenes, RefSeq 2018, and RefSeq 2020 libraries) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (k-mer read classifier run with SILVA, locally built Greengenes, Kraken Standard, and RefSeq-comparable libraries)

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Each of the 11 pairings yields a complete taxon-by-sample counts matrix in which rows are taxa and columns are all 136 mock community samples
- One CSV file is produced per pairing, and Kraken Standard results remain pairwise comparable with PathoScope RefSeq 2020 results
- Mothur and QIIME 2 classifiers were trained on the specific PCR-amplified 16S rRNA gene region rather than full-length references
- PathoScope SILVA, Greengenes, and RefSeq indices match the custom-configured libraries deposited by the authors rather than off-the-shelf releases
- Downstream genus- and species-level sensitivity/specificity/error scores reproduce the paper's ranking: PathoScope 2 and Kraken 2 outperform DADA2, QIIME 2, and Mothur; SILVA and RefSeq/Kraken Standard beat Greengenes

## Limitations

- Mock communities provide only a semblance of a 'ground truth' — technical bias and errors are introduced at every experimental step (pipetting of spike-in DNA, species-specific lysis difficulty, PCR amplification bias), so no pipeline can perfectly measure a mock community
- Contamination from reagents and the local environment (air, gloves, pipette tip boxes) complicates results
- RefSeq libraries can contain contamination; the authors recommend using older RefSeq libraries until newer versions are processed to remove contamination (e.g., E. coli 16S sequence found in T. flagellates RefSeq scaffolds)
- Results are time-stamped to Greengenes 13_8 and SILVA 138 — Greengenes2 (2022) and SILVA 138.1 were not part of the benchmark

## Evidence

- [methods] SILVA and Greengenes indices for Mothur and QIIME 2; Mothur with SILVA 138 and Greengenes 13_8 classifier training: [evidence span withheld — anchor status: not_found]
- [readme] Kraken 2 Greengenes index build command from README: "kraken2-build --db ${DBNAME} --special greengenes --threads 1"
- [readme] Kraken Standard comparability note from README: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
- [abstract] Mock community dataset scope from abstract: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [discussion] Reference library availability caveat from discussion: "we recommend using older RefSeq libraries until newer versions have been processed to remove contamination"
