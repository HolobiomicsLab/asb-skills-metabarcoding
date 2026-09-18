---
name: edna-timeseries-data-processing
description: Use when you have a per-locus taxa x time-point read-proportion table
  from eDNA metabarcoding (e.g. an 8-time-point Monterey Bay seawater series) and
  need a 0-1 relative abundance index per taxon that is robust to primer bias and
  consistent across multiple genetic loci.
when_to_use_negative:
- Input is raw sequencing reads or FASTQ data rather than a per-locus read-proportion
  table; upstream metabarcoding processing must be done first
- Data comes from a single locus and a single sample (no time series), since the within-taxon
  0-1 scaling is defined across a time series of read proportions
- The goal is absolute quantification of organismal abundance; the index only captures
  relative changes because amplification bias obscures the relationship between organismal
  and amplicon abundance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: pandas
  role: Loads the per-locus read-proportion table, performs the within-taxon division
    and per-taxon averaging of non-zero per-locus indices, and writes ensemble_abundance_index.csv
  repo: github.com/marinebon/eDNA_microbes_whales
- name: numpy
  role: Supports array-wise computation of the per-locus 0-1 scaled indices and the
    taxa x time-point ensemble matrix assembly
  repo: github.com/marinebon/eDNA_microbes_whales
- name: Python
  role: Implementation language for the abundance index reconstruction script
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:092aa7689d1374d55227099812188da5505cb6ba3473dfa4c36705d6278bde51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# edna-timeseries-data-processing

## Summary

Reconstructs the ensemble eDNA abundance index from a multilocus metabarcoding time-series: per-locus read proportions are scaled 0-1 within each taxon and averaged across loci to approximate relative changes in taxon abundance despite constant per-taxon amplification bias.

## When to use

Use when you have a per-locus taxa x time-point read-proportion table from eDNA metabarcoding (e.g. an 8-time-point Monterey Bay seawater series) and need a 0-1 relative abundance index per taxon that is robust to primer bias and consistent across multiple genetic loci.

## When NOT to use

- Input is raw sequencing reads or FASTQ data rather than a per-locus read-proportion table; upstream metabarcoding processing must be done first
- Data comes from a single locus and a single sample (no time series), since the within-taxon 0-1 scaling is defined across a time series of read proportions
- The goal is absolute quantification of organismal abundance; the index only captures relative changes because amplification bias obscures the relationship between organismal and amplicon abundance

## Inputs

- Per-locus read-proportion table (taxa x time points per locus) from upstream metabarcoding processing output, loadable with pandas
- Multilocus amplicon sequencing read-proportion data from an 18-month (2015-2016) Monterey Bay seawater eDNA time-series

## Outputs

- ensemble_abundance_index.csv: a taxa x 8 time-point matrix of ensemble abundance indices, all values in the 0-1 range

## How to apply

Load the per-locus read-proportion table (taxa x time points per locus) from the upstream metabarcoding processing output with pandas. For each locus, divide every taxon's read proportion at each time point by that taxon's maximum observed proportion across the time series, yielding per-locus relative indices in the 0-1 range. For each taxon and time point, average the per-locus indices retaining only non-zero values, as specified by the paper's ensemble index formula. Assemble the averaged values into a single taxa x 8 time-point matrix and write it to ensemble_abundance_index.csv. The rationale is that amplification bias arises from template-primer interaction that is constant across samples for a given taxon-primer pair, so within-taxon scaling removes that constant bias; averaging across loci increases the robustness of the index for taxa detected at multiple loci.

## Related tools

- **pandas** (Loads the per-locus read-proportion table, performs the within-taxon division and per-taxon averaging of non-zero per-locus indices, and writes ensemble_abundance_index.csv) — github.com/marinebon/eDNA_microbes_whales
- **numpy** (Supports array-wise computation of the per-locus 0-1 scaled indices and the taxa x time-point ensemble matrix assembly) — github.com/marinebon/eDNA_microbes_whales
- **Python** (Implementation language for the abundance index reconstruction script) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
idx = df.div(df.max(axis=1), axis=0); ensemble = idx.where(idx > 0).groupby(level=0).mean(); ensemble.to_csv('ensemble_abundance_index.csv')
```

## Evaluation signals

- Output CSV ensemble_abundance_index.csv exists and is readable by pandas
- Output contains one row per taxon and exactly one column per time point (8 time points)
- All index values fall within the 0-1 range (inclusive), since each per-locus index is a proportion of that taxon's maximum observed proportion
- Each taxon's index equals 1.0 at the time point where its raw read proportion was maximal for at least one contributing locus
- For taxa detected at multiple loci, the ensemble value equals the mean of the non-zero per-locus indices at each time point

## Limitations

- Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, so the index reflects amplicon rather than organismal abundance
- No method, including this one, identifies all taxa in an ecosystem, so absent taxa in the output reflect detection limits
- eDNA does not allow identification of life history stages and may include sequences from dead or decaying biological matter
- The method assumes amplification bias is constant across samples for a given taxon-primer pair; if bias varies with sample conditions the 0-1 scaling rationale breaks down

## Evidence

- [abstract] multilocus time-series input: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [discussion] primer bias rationale: "Amplification of DNA is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA"
- [intro] proportional abundance objective: "Here, we analyse a time-series of eDNA metabarcoding data to assess changes in the proportional abundance of taxa in a marine setting"
- [readme] repository purpose: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
