---
name: taxa-abundance-normalization
description: Use when multilocus eDNA metabarcoding read-proportion tables (taxa x
  time points per locus) need conversion into per-taxon relative abundance indices
  that are comparable across loci and robust to constant taxon-specific amplification
  bias.
when_to_use_negative:
- Input is raw amplicon reads or FASTQ files rather than a per-locus read-proportion
  (taxa x time point) table — upstream metabarcoding processing must be done first
- The goal is absolute organismal abundance estimation; the method only quantifies
  relative changes in proportional abundance because amplification bias obscures the
  organism-abundance/amplicon-abundance relationship
- A taxon is detected with only a single locus and its read proportions are dominated
  by stochastic detection noise rather than a stable taxon-primer bias
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pandas
  role: Loads the per-locus read-proportion table, performs the per-locus max-scaling
    and non-zero-averaging, and writes the taxa x 8 time-point ensemble index matrix
    to ensemble_abundance_index.csv
  repo: https://github.com/marinebon/eDNA_microbes_whales
- name: numpy
  role: Supports the element-wise division and non-zero mean computation used to build
    the per-locus and ensemble indices
- name: Python
  role: Implementation language for the abundance indexing workflow
  repo: https://github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:8d3114bfb1282582821a2a35953163655849cf91f294c8af16a684a00a4ffa23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# taxa-abundance-normalization

## Summary

A within-taxon, within-locus zero-to-one rescaling of eDNA amplicon read proportions that converts amplification-biased relative reads into comparable per-taxon relative abundance indices, averaged across genetic loci into an ensemble index. It quantifies relative changes in taxon abundance from multilocus metabarcoding despite primer bias.

## When to use

Use when multilocus eDNA metabarcoding read-proportion tables (taxa x time points per locus) need conversion into per-taxon relative abundance indices that are comparable across loci and robust to constant taxon-specific amplification bias.

## When NOT to use

- Input is raw amplicon reads or FASTQ files rather than a per-locus read-proportion (taxa x time point) table — upstream metabarcoding processing must be done first
- The goal is absolute organismal abundance estimation; the method only quantifies relative changes in proportional abundance because amplification bias obscures the organism-abundance/amplicon-abundance relationship
- A taxon is detected with only a single locus and its read proportions are dominated by stochastic detection noise rather than a stable taxon-primer bias

## Inputs

- Per-locus read-proportion table (taxa x time points per locus) from upstream metabarcoding processing, e.g. CSV loaded with pandas
- Multilocus eDNA amplicon sequencing read counts from the Monterey Bay 18-month (2015–2016) seawater time-series

## Outputs

- ensemble_abundance_index.csv: a taxa x 8 time-point matrix of ensemble relative abundance indices with all values in the 0–1 range

## How to apply

Load the per-locus read-proportion table (taxa x 8 time points) from the upstream metabarcoding processing output with pandas. For each locus independently, divide every taxon's read proportion at each time point by that taxon's maximum observed proportion across the time series, producing per-locus indices in the 0–1 range; this is valid because amplification bias from template–primer interaction is assumed constant across samples for a given taxon-primer pair. For each taxon and time point, average the per-locus indices retaining only non-zero values, following the paper's ensemble index formula. Assemble the averaged values into a single taxa x 8 time-point matrix and write it to ensemble_abundance_index.csv with pandas. Validate that the CSV exists, has one row per taxon and one column per time point, and that all values fall within the 0–1 range.

## Related tools

- **pandas** (Loads the per-locus read-proportion table, performs the per-locus max-scaling and non-zero-averaging, and writes the taxa x 8 time-point ensemble index matrix to ensemble_abundance_index.csv) — https://github.com/marinebon/eDNA_microbes_whales
- **numpy** (Supports the element-wise division and non-zero mean computation used to build the per-locus and ensemble indices)
- **Python** (Implementation language for the abundance indexing workflow) — https://github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; df = pd.read_csv('per_locus_read_proportions.csv'); idx = df.div(df.max(axis=1), axis=0); pd.DataFrame(idx).mean(axis=1).to_csv('ensemble_abundance_index.csv')
```

## Evaluation signals

- ensemble_abundance_index.csv exists and contains one row per taxon and one column per time point (taxa x 8 time-point matrix)
- All ensemble index values fall within the 0–1 range, as enforced by dividing each taxon by its own maximum per-locus proportion
- For each taxon, at least one time point attains the maximum value of 1 in each contributing locus before averaging
- Ensemble averaging retains only non-zero per-locus values, so no taxon-time-point cell is diluted toward zero by loci with zero detection
- Per-locus intermediate indices reproduce the original read proportions up to a per-taxon constant scaling factor (bias is assumed constant across samples per taxon-primer pair)

## Limitations

- Amplification of DNA is subject to substantial primer biases and is affected by inhibitors and variable abundances of template DNA, so indices are relative, not absolute, abundances
- eDNA does not allow identification of life history stages and may include sequences from dead or decaying biological matter
- No method, including this one, identifies all taxa in an ecosystem, so absent taxa cannot be distinguished from detection failure
- The ensemble average assumes the constant-bias model holds for every taxon-primer pair; taxa detected at only one locus get no cross-locus averaging

## Evidence

- [discussion] Amplification subject to primer biases, inhibitors and variable template abundances affecting taxa detection: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
- [intro] Objective: assess changes in proportional abundance of taxa: "Here, we analyse a time-series of eDNA metabarcoding data to assess changes in the proportional abundance of taxa in a marine setting"
- [abstract] Multilocus amplicon sequencing of an 18-month time-series: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
