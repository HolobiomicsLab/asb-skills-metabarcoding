---
name: pearson-r2-accuracy-evaluation
description: Use when benchmarking a taxonomic classifier or compositional estimator
  against known mock/simulated community ground truth and you need quantitative correlation
  (Pearson r2) and error (L1) metrics on abundance profiles.
when_to_use_negative:
- Samples with no ground-truth composition available (e.g., real environmental data)
  — r2/L1 cannot be computed without true abundances
- Analysis of samples with a high proportion of short reads, which MetaMaps does not
  handle well since reads below the minimum length remain unassigned
- Read-level assignment accuracy evaluation rather than composition estimation — this
  skill targets aggregate abundance metrics, not per-read precision/recall
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Primary compositional estimator; per-read assignments converted into sample
    composition via EM-based estimation; evaluated with r2/L1 metrics
  repo: https://github.com/DiltheyLab/MetaMaps
- name: Centrifuge
  role: Comparator tool whose per-read classifications are aggregated into taxon-level
    composition profiles for benchmarking
- name: Bracken
  role: Comparator tool whose per-read classifications are aggregated into taxon-level
    composition profiles for benchmarking
- name: LAST+MEGAN-LR
  role: Comparator long-read pipeline whose per-read classifications are aggregated
    into taxon-level composition profiles for benchmarking
- name: MEGAN
  role: Reference framework for taxonomic binning used in the comparison pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:534eefc7837a59660f5da26a16c3618ba95880f5160af6459f9650c9036f78e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Bracken
  - Centrifuge
  - LAST
  - LAST+MEGAN-LR
  - MEGAN
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pearson-r2-accuracy-evaluation

## Summary

Evaluates metagenomic sample-composition estimation accuracy by computing Pearson r2 and L1-distance between estimated and ground-truth taxonomic abundances for each tool-dataset combination, as done for MetaMaps versus Centrifuge, Bracken, and LAST+MEGAN-LR on i100, p25, CAMI, HMP7, and Zymo.

## When to use

Use when benchmarking a taxonomic classifier or compositional estimator against known mock/simulated community ground truth and you need quantitative correlation (Pearson r2) and error (L1) metrics on abundance profiles.

## When NOT to use

- Samples with no ground-truth composition available (e.g., real environmental data) — r2/L1 cannot be computed without true abundances
- Analysis of samples with a high proportion of short reads, which MetaMaps does not handle well since reads below the minimum length remain unassigned
- Read-level assignment accuracy evaluation rather than composition estimation — this skill targets aggregate abundance metrics, not per-read precision/recall

## Inputs

- Per-read taxonomic assignment files (e.g., MetaMaps classification_results.EM.reads2Taxon; Centrifuge/Bracken/LAST+MEGAN-LR classification outputs) for i100, p25, CAMI, HMP7, Zymo
- MetaMaps .EM.WIMP sample-composition file (Absolute, EMFrequency, PotFrequency columns)
- Ground-truth composition tables for the simulated (i100, p25) and mock (HMP7, Zymo, CAMI) communities

## Outputs

- composition_metrics.csv — full table of Pearson r2 and L1-distance per tool-dataset combination
- composition_profiles.csv — per-tool estimated taxonomic composition profiles

## How to apply

Load per-read taxonomic assignment files for each tool (e.g., MetaMaps, Centrifuge, Bracken, LAST+MEGAN-LR) on the i100, p25, CAMI, HMP7, and Zymo datasets. Convert MetaMaps per-read assignments into composition estimates using its EM-based estimation of overall sample composition (EMFrequency/PotFrequency columns of the .EM.WIMP output, with 'definedGenomes' as strain-level resolution), and aggregate the per-read classifications of the other tools into taxon-level composition profiles. Match estimated compositions against ground truth at strain level for i100 and p25 and at species level for HMP7, Zymo, and CAMI, then compute Pearson r2 and L1-distance for every tool-dataset combination. Validate against the paper's reported values: strain-level r2 of 0.88 (i100) and 0.78 (p25), species-level r2 of 0.98 (HMP7) and 0.97 (Zymo), and r2 > 0.97 at higher taxonomic levels.

## Related tools

- **MetaMaps** (Primary compositional estimator; per-read assignments converted into sample composition via EM-based estimation; evaluated with r2/L1 metrics) — https://github.com/DiltheyLab/MetaMaps
- **Centrifuge** (Comparator tool whose per-read classifications are aggregated into taxon-level composition profiles for benchmarking)
- **Bracken** (Comparator tool whose per-read classifications are aggregated into taxon-level composition profiles for benchmarking)
- **LAST+MEGAN-LR** (Comparator long-read pipeline whose per-read classifications are aggregated into taxon-level composition profiles for benchmarking)
- **MEGAN** (Reference framework for taxonomic binning used in the comparison pipeline)

## Examples

```
./metamaps classify --mappings classification_results --DB databases/miniSeq+H  # then compute Pearson r2/L1 on classification_results.EM.WIMP vs. ground truth
```

## Evaluation signals

- Reproduced strain-level Pearson r2 equals 0.88 for i100 and 0.78 for p25 (MetaMaps)
- Species-level Pearson r2 reaches 0.98 on HMP7 and 0.97 on Zymo, consistent with the reported r2 > 0.97
- r2 increases to >0.99 when compositions are aggregated above strain level
- composition_metrics.csv contains one r2/L1 row for every tool-dataset combination and composition_profiles.csv matches the taxon sets of each tool's estimated profiles
- L1-distance values rank tools consistently with the r2 ordering reported in Fig. 2, Fig. 5, and Supplementary Data 3

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations, biasing composition estimates for short-read-rich samples
- Complex samples comprising many non-characterized entities remain challenging, and database-sample incongruity (presence of yet-uncharacterized taxonomic entities) can distort r2/L1 comparisons
- Metrics are only meaningful at the taxonomic level where ground truth exists (strain level for i100/p25; species level for HMP7, Zymo, CAMI)

## Evidence

- [results] At the strain level, MetaMaps achieves a Pearson r2 between estimated and true abundances of 0.88 (i100) and 0.78 (p25): [evidence span withheld — anchor status: not_found]
- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [discussion] EM algorithm for estimation of overall sample composition: "the application of the EM algorithm for estimation of overall sample composition"
- [readme] The level 'definedGenomes' represents strain-level resolution; the EM algorithm is carried out at this level: [evidence span withheld — anchor status: not_found]
- [discussion] Reads below minimum length end up with minimizer sets too small to reliably determine mapping locations: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
