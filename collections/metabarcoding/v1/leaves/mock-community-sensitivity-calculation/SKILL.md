---
name: mock-community-sensitivity-calculation
description: Use when benchmarking taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope
  2, Kraken 2) or reference libraries (SILVA 138, Greengenes 13_8, Kraken Standard,
  RefSeq) on 16S mock community data with known composition.
when_to_use_negative:
- Data comes from real (non-mock) microbiome samples with no ground-truth composition
  table — sensitivity/specificity against ground truth cannot be computed
- Analysis goal is producing taxonomy assignments rather than benchmarking profiler
  accuracy — use the classification pipelines directly instead
- Input is already the aggregated per-sample metric table — proceed to aggregation/tabulation
  rather than recomputing per-taxon detection
edam_operation: http://edamontology.org/operation_0369
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: PathoScope 2
  role: Profiler evaluated; produces per-sample taxon counts matrices for sensitivity/specificity
    computation
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Profiler evaluated; produces per-sample taxon counts matrices for sensitivity/specificity
    computation
  repo: github.com/aubreyodom/16SBenchmarking
- name: DADA2
  role: Profiler evaluated; QIIME 2 DADA2 plugin outputs are collapsed and scored
    against ground truth
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: Profiler evaluated; outputs collapsed to genus/species level for benchmarking
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Profiler evaluated; outputs collapsed to genus/species level for benchmarking
  repo: github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: Provides download_refseq() for obtaining up-to-date RefSeq reference library
    indices used in the pairings
  repo: compbiomed.github.io/metascope-docs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:8d8280c192e85dcac5ba8474e80c35e758641a443b7f05aabc25ab059508f3cf
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
  edam_operation: http://edamontology.org/operation_0369
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mock-community-sensitivity-calculation

## Summary

Computes genus- and species-level taxon detection sensitivity and read assignment specificity by comparing profiler/library outputs against mock community ground truth, enabling benchmarking of 16S amplicon classification pipelines.

## When to use

Use when benchmarking taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) or reference libraries (SILVA 138, Greengenes 13_8, Kraken Standard, RefSeq) on 16S mock community data with known composition.

## When NOT to use

- Data comes from real (non-mock) microbiome samples with no ground-truth composition table — sensitivity/specificity against ground truth cannot be computed
- Analysis goal is producing taxonomy assignments rather than benchmarking profiler accuracy — use the classification pipelines directly instead
- Input is already the aggregated per-sample metric table — proceed to aggregation/tabulation rather than recomputing per-taxon detection

## Inputs

- Per-pairing taxon counts matrices from DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 classification runs
- Mock community ground-truth composition tables (136 mock samples)
- Taxonomic hierarchy mapping for collapsing counts to genus and species levels

## Outputs

- Per-sample taxon detection sensitivity values at genus and species levels
- Per-sample read assignment specificity values
- Summary table of means and standard deviations per profiler-library pairing (matching Figure 2A-D and Supplementary Tables S2-S5)

## How to apply

Load the per-pairing taxon counts matrices produced by each profiler run (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) against each reference library (SILVA 138, Greengenes 13_8, Kraken Standard, RefSeq), along with the mock community ground-truth composition tables for the 136 mock samples. Collapse each counts matrix to genus-level and species-level rows, tracing taxonomy back through the taxonomic hierarchy for profilers that report at finer or coarser ranks (PathoScope, QIIME 2, and Mothur all allow tracing back the taxonomic hierarchy). For each sample and pairing, compute taxon detection sensitivity as the fraction of ground-truth taxa present at >=0.1% relative abundance that are detected by the profiler, and read assignment specificity as the fraction of reads assigned to taxa expected in the ground truth. Aggregate per-sample values into means and standard deviations for each profiler-library pairing at genus and species levels, tabulating results to match Figure 2A-D and Supplementary Tables S2-S5.

## Related tools

- **PathoScope 2** (Profiler evaluated; produces per-sample taxon counts matrices for sensitivity/specificity computation) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Profiler evaluated; produces per-sample taxon counts matrices for sensitivity/specificity computation) — github.com/aubreyodom/16SBenchmarking
- **DADA2** (Profiler evaluated; QIIME 2 DADA2 plugin outputs are collapsed and scored against ground truth) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (Profiler evaluated; outputs collapsed to genus/species level for benchmarking) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Profiler evaluated; outputs collapsed to genus/species level for benchmarking) — github.com/aubreyodom/16SBenchmarking
- **MetaScope** (Provides download_refseq() for obtaining up-to-date RefSeq reference library indices used in the pairings) — compbiomed.github.io/metascope-docs

## Examples

```
sens = sum(1 for t,ab in truth.items() if ab >= 0.001 and t in detected) / sum(1 for ab in truth.values() if ab >= 0.001)
```

## Evaluation signals

- Summary table reproduces reported benchmarks: PathoScope+RefSeq2018 genus sensitivity mean equals 0.88 (SD = 0.14)
- Tabulated means/SDs match Figure 2A-D and Supplementary Tables S2-S5 values within rounding tolerance
- Sensitivity denominators include only ground-truth taxa present at >=0.1% relative abundance
- Read assignment specificity computed as the fraction of reads assigned to taxa expected in the mock community ground truth
- Genus- and species-level collapses trace taxa back through the taxonomic hierarchy for profilers reporting at other ranks

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no pipeline can theoretically be perfectly measured
- Mock species relative abundances may be affected by subtle pipetting variations when aliquoting spike-in DNA
- Species vary in lysis difficulty, causing underrepresentation or absence in collected cDNA libraries
- Amplification bias occurs during PCR despite universal primer design, and contamination from reagents and the local environment (air, gloves, pipette tip boxes) can further complicate results

## Evidence

- [full_text] compute taxon detection sensitivity as the fraction of ground-truth taxa present at >=0.1% relative abundance that are detected by the profiler: [evidence span withheld — anchor status: not_found]
- [full_text] compute read assignment specificity as the fraction of reads assigned to taxa expected in the mock community ground truth: [evidence span withheld — anchor status: not_found]
- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [abstract] PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur: "PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur"
- [discussion] PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
