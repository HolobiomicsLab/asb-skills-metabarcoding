---
name: l1-distance-compositional-comparison
description: Use when benchmarking metagenomic classification/composition-estimation
  tools against known sample ground truth, and you need a per-tool, per-dataset L1-distance
  metric between estimated and true taxonomic abundance profiles.
when_to_use_negative:
- Sample genomes or close relatives are not in the reference database, since database-sample
  mismatch invalidates the composition comparison (as discussed for HMP7 and CAMI)
- The sample contains a high proportion of short reads or many non-characterized entities,
  which MetaMaps and this benchmarking design do not handle well
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Produces EM-based sample composition estimates and per-read taxonomic assignments
    that are converted into composition profiles for L1/r2 evaluation
  repo: https://github.com/DiltheyLab/MetaMaps
- name: Centrifuge
  role: Per-read classifications aggregated into taxon-level composition profiles
    as a comparison tool
- name: Bracken
  role: Per-read classifications aggregated into taxon-level composition profiles
    as a comparison tool
- name: MEGAN
  role: LAST+MEGAN-LR per-read classifications aggregated into taxon-level composition
    profiles as a comparison tool
- name: LAST
  role: Aligner whose output feeds LAST+MEGAN-LR classifications used as a comparison
    profile
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:b15bf6a3f50d2764fdf1e30dff448e528d468c622d1b2ba2733021b931a03464
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Bracken
  - Centrifuge
  - LAST
  - MEGAN
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# l1-distance-compositional-comparison

## Summary

Quantify the accuracy of metagenomic sample-composition estimates by computing the L1 distance between tool-estimated taxonomic abundance profiles and ground-truth compositions, alongside Pearson r2, for long-read metagenomic classifiers such as MetaMaps on the i100, p25, CAMI, HMP7, and Zymo datasets.

## When to use

Use when benchmarking metagenomic classification/composition-estimation tools against known sample ground truth, and you need a per-tool, per-dataset L1-distance metric between estimated and true taxonomic abundance profiles.

## When NOT to use

- Sample genomes or close relatives are not in the reference database, since database-sample mismatch invalidates the composition comparison (as discussed for HMP7 and CAMI)
- The sample contains a high proportion of short reads or many non-characterized entities, which MetaMaps and this benchmarking design do not handle well

## Inputs

- Per-read taxonomic assignment files for MetaMaps, Centrifuge, Bracken, and LAST+MEGAN-LR (e.g. MetaMaps classification_results.EM.reads2Taxon and .EM.WIMP files)
- Ground-truth sample compositions for i100 and p25 (strain level) and HMP7, Zymo, and CAMI (species level)

## Outputs

- composition_metrics.csv: Pearson r2 and L1-distance for every tool-dataset-taxonomic-level combination
- composition_profiles.csv: per-tool estimated taxonomic composition profiles

## How to apply

Convert MetaMaps per-read assignments into sample composition estimates using its EM-based estimation of overall sample composition (PotFrequency from the .EM.WIMP output), and aggregate the per-read classifications of comparison tools (Centrifuge, Bracken, LAST+MEGAN-LR) into taxon-level composition profiles. Match estimated compositions against ground truth at strain level for the simulated i100 and p25 experiments and at species level for HMP7, Zymo, and CAMI. Compute the L1 distance (sum of absolute differences between estimated and true abundances) and the Pearson r2 for every tool-dataset combination; interpret low L1 with high r2 (e.g. strain-level r2 of 0.88 for i100 and 0.78 for p25, rising to >0.99 at higher taxonomic levels) as agreement with reported benchmarks. Write the full metrics table to composition_metrics.csv and the per-tool estimated compositions to composition_profiles.csv so metrics can be cross-checked against Fig. 2, Fig. 5, and Supplementary Data 3.

## Related tools

- **MetaMaps** (Produces EM-based sample composition estimates and per-read taxonomic assignments that are converted into composition profiles for L1/r2 evaluation) — https://github.com/DiltheyLab/MetaMaps
- **Centrifuge** (Per-read classifications aggregated into taxon-level composition profiles as a comparison tool)
- **Bracken** (Per-read classifications aggregated into taxon-level composition profiles as a comparison tool)
- **MEGAN** (LAST+MEGAN-LR per-read classifications aggregated into taxon-level composition profiles as a comparison tool)
- **LAST** (Aligner whose output feeds LAST+MEGAN-LR classifications used as a comparison profile)

## Examples

```
./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Reproduced strain-level Pearson r2 matches the reported values: 0.88 for i100 and 0.78 for p25
- Reproduced species-level r2 matches the reported values: 0.98 for HMP7 and 0.97 for Zymo, and r2 > 0.97 overall on simulated and real data when close relatives are in the database
- r2 values increase to >0.99 when compositions are aggregated to higher (above-strain) taxonomic levels
- Metrics in composition_metrics.csv are consistent with Fig. 2, Fig. 5, and Supplementary Data 3 of the paper
- MetaMaps composition estimates use PotFrequency (final frequency after correcting for unmapped reads) from the .EM.WIMP file, not raw read counts

## Limitations

- MetaMaps assigns no taxonomy to reads below the minimum length requirement, since their minimizer sets are too small to reliably determine mapping locations
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- Complex samples comprising many non-characterized entities remain challenging; database-sample incongruity (presence of yet-uncharacterized taxonomic entities) is an important unresolved concern
- The MetaMaps '--maxmemory' memory target is implemented heuristically; actual memory usage will fluctuate and exceed the target

## Evidence

- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for estimation of sample composition on simulated and real data when sample genomes or close relatives are in the database: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [discussion] EM algorithm estimation of overall sample composition: "the application of the EM algorithm for estimation of overall sample composition"
- [readme] WIMP output columns define the composition estimate: [evidence span withheld — anchor status: not_found]
- [discussion] Dataset-sample mismatch discussion motivating care with ground-truth matching: "As we have shown in the HMP7 and CAMI experiments, visual analysis or automatic thresholding can contribute to detecting and dealing with database-sample mismatches."
