---
name: count-matrix-aggregation-to-taxon-levels
description: Use when profiler outputs report taxa at finer or coarser ranks than
  genus/species and per-pairing genus- and species-level sensitivity/specificity must
  be benchmarked.
when_to_use_negative:
- Input is already collapsed to the target rank (e.g. an OTU/ASV table already aggregated
  to genus), so re-tracing the hierarchy would double-count or misassign reads
- The goal is amplicon sequence variant inference or read preprocessing rather than
  benchmarking taxon-level classification output
- Ground-truth composition tables for the samples are unavailable, since sensitivity
  and specificity cannot be computed without them
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: PathoScope 2
  role: Profiler whose output counts are collapsed via tracing back the taxonomic
    hierarchy of a given microbe
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Profiler whose output counts are collapsed to genus/species rows for sensitivity
    and specificity computation
  repo: github.com/aubreyodom/16SBenchmarking
- name: DADA2
  role: Profiler whose QIIME 2-plugin output counts are collapsed to genus and species
    levels
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: Profiler whose output counts are collapsed; supports tracing back the taxonomic
    hierarchy
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Profiler whose output counts are collapsed; supports tracing back the taxonomic
    hierarchy
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:423b3fd2b37a7523e47d14fc7084bd64adcc16fcb45f2cf881236fb7102b719c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Count matrix aggregation to taxon levels

## Summary

Collapses profiler taxon count matrices (from DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) to genus-level and species-level rows so detection sensitivity and assignment specificity can be computed against mock community ground truth.

## When to use

Use when profiler outputs report taxa at finer or coarser ranks than genus/species and per-pairing genus- and species-level sensitivity/specificity must be benchmarked.

## When NOT to use

- Input is already collapsed to the target rank (e.g. an OTU/ASV table already aggregated to genus), so re-tracing the hierarchy would double-count or misassign reads
- The goal is amplicon sequence variant inference or read preprocessing rather than benchmarking taxon-level classification output
- Ground-truth composition tables for the samples are unavailable, since sensitivity and specificity cannot be computed without them

## Inputs

- Per-pairing taxon counts matrices from DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 runs
- Reference library assignments for SILVA 138, Greengenes 13_8, Kraken Standard, and RefSeq (2018/2020)
- Mock community ground-truth composition tables (136 samples)

## Outputs

- Genus-level collapsed counts matrices per profiler-library pairing
- Species-level collapsed counts matrices per profiler-library pairing
- Per-pairing means and SDs of taxon detection sensitivity and read assignment specificity (Figure 2A-D, Supplementary Tables S2-S5)

## How to apply

Load the per-pairing taxon counts matrices produced by DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 classification runs against the SILVA 138, Greengenes 13_8, Kraken Standard, and RefSeq libraries, along with the 136-sample mock community ground-truth composition tables. Collapse each counts matrix to genus-level and species-level taxonomic rows, tracing taxonomy back through the taxonomic hierarchy as needed for profilers that report at finer or coarser ranks — PathoScope, QIIME 2, and Mothur all allow tracing back the taxonomic hierarchy of a given microbe. After collapsing, compute taxon detection sensitivity as the fraction of ground-truth taxa present at >=0.1% relative abundance that are detected, and read assignment specificity as the fraction of reads assigned to taxa expected in the mock community. Aggregate per-sample values into means and standard deviations for each profiler-library pairing at genus and species levels to reproduce the Figure 2A-D and Supplementary Tables S2-S5 summaries.

## Related tools

- **PathoScope 2** (Profiler whose output counts are collapsed via tracing back the taxonomic hierarchy of a given microbe) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Profiler whose output counts are collapsed to genus/species rows for sensitivity and specificity computation) — github.com/aubreyodom/16SBenchmarking
- **DADA2** (Profiler whose QIIME 2-plugin output counts are collapsed to genus and species levels) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (Profiler whose output counts are collapsed; supports tracing back the taxonomic hierarchy) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Profiler whose output counts are collapsed; supports tracing back the taxonomic hierarchy) — github.com/aubreyodom/16SBenchmarking

## Examples

```
Rscript aggregate_taxon_counts.R --profiler pathoscope --library refseq2018 --ranks genus,species --ground-truth mock_composition.tsv --out genus_species_counts/
```

## Evaluation signals

- Summary table reproduces the reported benchmark, e.g. PathoScope+RefSeq2018 genus detection sensitivity mean equals 0.88 (SD = 0.14)
- Tabulated per-pairing means/SDs match Figure 2A-D and Supplementary Tables S2-S5 values within rounding tolerance
- All 136 mock samples are represented in the aggregated genus- and species-level matrices
- Every collapsed row resolves to a valid genus or species rank after tracing the taxonomic hierarchy, with no unassigned leftover rows
- Detection sensitivity denominators use only ground-truth taxa present at >=0.1% relative abundance

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no pipeline can be perfectly measured
- Profilers reporting at finer or coarser ranks require hierarchical tracing, which can misattribute reads when lineage annotations are inconsistent across reference libraries
- Reference libraries may contain contamination (e.g. T. flagellates scaffolds carrying E. coli 16S sequence in RefSeq), inflating false assignments after collapse

## Evidence

- [other] Collapse each counts matrix to genus-level and species-level taxonomic rows, tracing taxonomy back through the taxonomic hierarchy as needed for profilers that report at finer or coarser ranks.: [evidence span withheld — anchor status: not_found]
- [discussion] PathoScope, QIIME 2, and Mothur all allow tracing back the taxonomic hierarchy of a given microbe.: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
- [other] compute taxon detection sensitivity as the fraction of ground-truth taxa present at >=0.1% relative abundance that are detected by the profiler: [evidence span withheld — anchor status: not_found]
- [other] PathoScope was the most sensitive method at detecting genera, peaking when paired with the RefSeq2018 reference library (mean = 0.88, SD = 0.14).: [evidence span withheld — anchor status: not_found]
- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
