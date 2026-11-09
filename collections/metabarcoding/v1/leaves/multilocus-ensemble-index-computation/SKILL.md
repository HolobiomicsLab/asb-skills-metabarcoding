---
name: multilocus-ensemble-index-computation
description: Use when you have taxa x time-point read-proportion tables from multilocus
  eDNA amplicon sequencing and need relative abundance changes comparable across loci
  despite amplification bias.
when_to_use_negative:
- The input is already a normalized feature table or relative abundance matrix rather
  than per-locus raw read proportions — re-scaling would double-normalize
- A taxon is detected at only one locus — no ensemble averaging is possible, and the
  per-locus index alone is the output
- You need absolute organismal abundance estimates; amplification bias from template–primer
  interaction means amplicon abundance is only proportional, not quantitative
edam_topics: []
tools:
- name: pandas
  role: Loads the per-locus read-proportion tables, computes the max-scaled per-locus
    indices and non-zero ensemble averages, and writes ensemble_abundance_index.csv
- name: numpy
  role: Array arithmetic for the per-locus max-scaling and masked (non-zero) averaging
    across loci
- name: Python
  role: Implementation language for the index computation script
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:546017d6007074b12f24eae1ba0ed6af52f2fa406e4bcdb637828dd449229c3e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# multilocus-ensemble-index-computation

## Summary

Computes a within-taxon, within-locus relative abundance index (0–1) from eDNA amplicon read-proportion tables and, when a taxon is detected at multiple genetic loci, averages its per-locus indices into an ensemble index. This mitigates primer/amplification bias that obscures the relationship between organismal and amplicon abundance in metabarcoding time-series.

## When to use

Use when you have taxa x time-point read-proportion tables from multilocus eDNA amplicon sequencing and need relative abundance changes comparable across loci despite amplification bias.

## When NOT to use

- The input is already a normalized feature table or relative abundance matrix rather than per-locus raw read proportions — re-scaling would double-normalize
- A taxon is detected at only one locus — no ensemble averaging is possible, and the per-locus index alone is the output
- You need absolute organismal abundance estimates; amplification bias from template–primer interaction means amplicon abundance is only proportional, not quantitative

## Inputs

- Per-locus read-proportion table (taxa x 8 time points) from multilocus eDNA amplicon sequencing of the Monterey Bay 2015–2016 seawater time-series (CSV/TSV readable by pandas)
- Mapping of each taxon to the genetic loci at which it was detected

## Outputs

- ensemble_abundance_index.csv — a taxa x 8 time-point matrix of ensemble indices with all values in the 0–1 range

## How to apply

Load the per-locus read-proportion table (taxa x time points per locus) from the upstream metabarcoding output with pandas. For each locus, divide every taxon's read proportion at each of the 8 time points by that taxon's maximum observed proportion across the time series, yielding per-locus relative indices in the 0–1 range; this assumes the amplification bias from template–primer interaction is constant across samples for a given taxon-primer pair. For each taxon and time point, average the per-locus indices retaining only non-zero values, following the paper's ensemble index formula. Assemble the averaged values into a single taxa x 8 time-point matrix and write it to ensemble_abundance_index.csv with pandas.

## Related tools

- **pandas** (Loads the per-locus read-proportion tables, computes the max-scaled per-locus indices and non-zero ensemble averages, and writes ensemble_abundance_index.csv)
- **numpy** (Array arithmetic for the per-locus max-scaling and masked (non-zero) averaging across loci)
- **Python** (Implementation language for the index computation script) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; df = pd.read_csv('per_locus_read_proportions.csv', index_col=0); idx = df.div(df.max(axis=1), axis=0); ensemble = idx.replace(0, pd.NA).mean(axis=1); ensemble.to_csv('ensemble_abundance_index.csv')
```

## Evaluation signals

- ensemble_abundance_index.csv exists and contains exactly one row per taxon and one column per time point (8 columns)
- Every ensemble index value falls within the 0–1 inclusive range
- Taxa whose read proportion was maximal at a time point have index exactly 1.0 at that time point for every locus where they occur
- Ensemble values for multi-locus taxa equal the mean of the non-zero per-locus indices, and zero-value (undetected) loci are excluded from the mean rather than counted as zero

## Limitations

- Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, so indices track relative amplicon proportions rather than true organismal abundance
- The method assumes the taxon-primer amplification bias is constant across samples for a given locus; departures from this invalidate the max-scaling
- No method, including this one, identifies all taxa in an ecosystem, so the index covers only detected taxonomic groups

## Evidence

- [discussion] Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
- [abstract] Multilocus amplicon sequencing of an eighteen-month seawater time-series: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [intro] Objective: assess changes in proportional abundance of taxa: "Here, we analyse a time-series of eDNA metabarcoding data to assess changes in the proportional abundance of taxa in a marine setting"
- [readme] README purpose of the repository: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
