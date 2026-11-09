---
name: relative-abundance-aggregation-across-loci
description: Use when a taxa x time-point read-proportion table from multilocus eDNA
  amplicon sequencing contains taxa detected at several genetic loci, and you need
  an amplification-bias-insensitive relative abundance index per taxon per time point.
when_to_use_negative:
- Input is already a normalized, zero-to-one-scaled relative abundance index — re-applying
  the within-taxon max division would flatten true abundance variation.
- Each taxon is detected by only a single locus, so there is nothing to ensemble-average;
  use the per-locus index directly.
- 'The goal is absolute organismal abundance: the index captures only proportional
  changes, since amplification biases and variable template abundances mean read proportions
  do not equal organismal abundance.'
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3959
- http://edamontology.org/topic_3053
tools:
- name: pandas
  role: Load the per-locus read-proportion table, compute the per-locus max-scaled
    indices, average non-zero per-locus values, and write ensemble_abundance_index.csv
- name: numpy
  role: Numerical masking of zero values and array-wise averaging across loci when
    building the ensemble index
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:6f022c1241287b0a384f1f629b3b85dbb05a1aff9d4ad20454188e28a677b5d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3959
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# relative-abundance-aggregation-across-loci

## Summary

Constructs an ensemble per-taxon abundance index from multilocus eDNA metabarcoding time-series data by within-taxon, within-locus zero-to-one scaling of read proportions followed by averaging across loci. This mitigates taxon-specific amplification bias so proportional abundance changes across the 8 Monterey Bay sampling time points can be compared.

## When to use

Use when a taxa x time-point read-proportion table from multilocus eDNA amplicon sequencing contains taxa detected at several genetic loci, and you need an amplification-bias-insensitive relative abundance index per taxon per time point.

## When NOT to use

- Input is already a normalized, zero-to-one-scaled relative abundance index — re-applying the within-taxon max division would flatten true abundance variation.
- Each taxon is detected by only a single locus, so there is nothing to ensemble-average; use the per-locus index directly.
- The goal is absolute organismal abundance: the index captures only proportional changes, since amplification biases and variable template abundances mean read proportions do not equal organismal abundance.

## Inputs

- Per-locus read-proportion table (taxa x 8 time points per locus) from upstream eDNA metabarcoding processing output
- Locus identifier mapping linking each taxon to the genetic loci at which it was detected

## Outputs

- ensemble_abundance_index.csv: taxa x 8 time-point matrix of ensemble relative abundance indices scaled 0-1

## How to apply

Load the per-locus read-proportion table (taxa x 8 time points per locus) from the upstream metabarcoding processing output with pandas. For each locus, divide each taxon's read proportion at each time point by that taxon's maximum observed proportion across the time series, producing per-locus indices in the 0-1 range; this within-taxon scaling assumes amplification bias from template-primer interaction is constant across samples for a given taxon-primer pair. For each taxon and time point, average the per-locus indices retaining only non-zero values, per the paper's ensemble index formula, so detection at one locus is not diluted by zeros at another. Assemble the result into a single taxa x 8 time-point matrix and write it to ensemble_abundance_index.csv with pandas. Validate that the CSV has one row per taxon, one column per time point, and all values within 0-1.

## Related tools

- **pandas** (Load the per-locus read-proportion table, compute the per-locus max-scaled indices, average non-zero per-locus values, and write ensemble_abundance_index.csv)
- **numpy** (Numerical masking of zero values and array-wise averaging across loci when building the ensemble index)

## Examples

```
import pandas as pd; df = pd.read_csv('per_locus_read_proportions.csv', index_col=0); idx = df.div(df.max(axis=1), axis=0); idx.to_csv('ensemble_abundance_index.csv')
```

## Evaluation signals

- Output file ensemble_abundance_index.csv exists and contains exactly one row per taxon and one column per time point (8 time-point columns).
- Every ensemble index value falls within the closed interval 0-1.
- For each taxon and time point, the ensemble value equals the mean of only the non-zero per-locus indices, so a zero appears only when all loci report zero (non-detection).
- Each taxon's maximum ensemble index across the 8 time points equals 1 for at least one time point (by construction of the within-taxon max scaling), unless the taxon is never detected.
- Spot-check one taxon-locus pair: dividing its raw read proportions by its time-series maximum reproduces the stored per-locus index.

## Limitations

- The index rests on the assumption that amplification bias from template-primer interaction is constant across samples for a given taxon-primer pair; primer biases, inhibitors, and variable template abundances can still affect taxon detection.
- Averaging per-locus indices assumes per-locus indices are exchangeable for a taxon; taxa detected at few loci have noisier ensemble indices.
- The index reflects proportional, not absolute, abundance and cannot identify life-history stages or exclude sequences from dead or decaying biological matter.
- No method, including this one, identifies all taxa in an ecosystem, so the index is limited to detected taxonomic groups (Family or higher rank in this study).

## Evidence

- [other] task_006 workflow: [evidence span withheld — anchor status: not_found]
- [intro] Djurhuus et al. 2020 intro: "we analyse a time-series of eDNA metabarcoding data to assess changes in the proportional abundance of taxa in a marine setting"
- [discussion] Djurhuus et al. 2020 discussion: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
- [readme] marinebon/eDNA_microbes_whales README: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
