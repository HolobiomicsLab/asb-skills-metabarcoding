---
name: per-read-taxonomic-assignment-aggregation
description: Use when per-read taxonomic assignment files (e.g. MetaMaps reads2Taxon/WIMP
  output, or Centrifuge/Bracken/MEGAN-LR classifications) on simulated i100, p25,
  CAMI, HMP7, or Zymo long-read datasets must be turned into composition profiles
  and validated via Pearson r2 and L1 distance.
when_to_use_negative:
- Input is already a sample composition/abundance table rather than per-read assignments
- The sample contains a high proportion of short reads, which MetaMaps is not designed
  to handle
- The sample comprises many non-characterized entities absent from the reference database,
  where composition estimation is unreliable
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MetaMaps
  role: Produces per-read assignments and EM-based sample composition estimation;
    its .EM.WIMP/.EM.reads2Taxon outputs are the source composition profiles
  repo: github.com/DiltheyLab/MetaMaps
- name: Centrifuge
  role: Provides per-read classifications that are aggregated into taxon-level composition
    profiles for comparison
- name: Bracken
  role: Provides per-read classifications that are aggregated into taxon-level composition
    profiles for comparison
- name: MEGAN
  role: Used with LAST (LAST+MEGAN-LR) to produce per-read classifications aggregated
    into composition profiles
- name: LAST
  role: Aligner whose mappings, via MEGAN-LR, provide per-read classifications for
    aggregation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:c5f68f9cfeb06c9a561d74881ccbb28078f842aa9b4286fc806e78ce5bbfe793
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
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# per-read-taxonomic-assignment-aggregation

## Summary

Converts per-read taxonomic assignments from long-read metagenomic classifiers into sample composition estimates — using MetaMaps' EM-based estimation for its own assignments and taxon-level aggregation of per-read classifications for Centrifuge, Bracken, and LAST+MEGAN-LR — so composition accuracy can be scored against ground truth.

## When to use

Use when per-read taxonomic assignment files (e.g. MetaMaps reads2Taxon/WIMP output, or Centrifuge/Bracken/MEGAN-LR classifications) on simulated i100, p25, CAMI, HMP7, or Zymo long-read datasets must be turned into composition profiles and validated via Pearson r2 and L1 distance.

## When NOT to use

- Input is already a sample composition/abundance table rather than per-read assignments
- The sample contains a high proportion of short reads, which MetaMaps is not designed to handle
- The sample comprises many non-characterized entities absent from the reference database, where composition estimation is unreliable

## Inputs

- Per-read taxonomic assignment files for MetaMaps (e.g. .EM.reads2Taxon, .EM.WIMP output)
- Per-read classifications from Centrifuge, Bracken, and LAST+MEGAN-LR on i100, p25, CAMI, HMP7, and Zymo datasets
- Ground-truth sample compositions (strain level for i100/p25; species level for HMP7, Zymo, CAMI)

## Outputs

- composition_metrics.csv — Pearson r2 and L1-distance for every tool–dataset combination
- composition_profiles.csv — per-tool estimated sample compositions

## How to apply

Load the per-read taxonomic assignment files for MetaMaps, Centrifuge, Bracken, and LAST+MEGAN-LR on the i100, p25, CAMI, HMP7, and Zymo datasets. For MetaMaps, convert its per-read assignments into sample composition estimates using its EM-based estimation of overall sample composition (the .EM.WIMP file, where the 'definedGenomes' level represents strain-level resolution and PotFrequency is the final estimated frequency after correcting for unmapped reads); for Centrifuge, Bracken, and LAST+MEGAN-LR, aggregate their per-read classifications into taxon-level composition profiles. Match estimated compositions against ground truth at strain level for i100 and p25 and at species level for HMP7, Zymo, and CAMI. Compute Pearson r2 and L1-distance between estimated and true compositions for every tool–dataset combination, and validate the reproduced metrics against reported values (strain-level r2 of 0.88 for i100 and 0.78 for p25; species-level r2 of 0.98 for HMP7 and 0.97 for Zymo; >0.99 at higher taxonomic levels). Write the full metrics table to composition_metrics.csv and the per-tool estimated compositions to composition_profiles.csv.

## Related tools

- **MetaMaps** (Produces per-read assignments and EM-based sample composition estimation; its .EM.WIMP/.EM.reads2Taxon outputs are the source composition profiles) — github.com/DiltheyLab/MetaMaps
- **Centrifuge** (Provides per-read classifications that are aggregated into taxon-level composition profiles for comparison)
- **Bracken** (Provides per-read classifications that are aggregated into taxon-level composition profiles for comparison)
- **MEGAN** (Used with LAST (LAST+MEGAN-LR) to produce per-read classifications aggregated into composition profiles)
- **LAST** (Aligner whose mappings, via MEGAN-LR, provide per-read classifications for aggregation)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Strain-level Pearson r2 reproduces 0.88 for i100 and 0.78 for p25, rising to >0.99 at higher taxonomic levels
- Species-level Pearson r2 reproduces 0.98 for HMP7 and 0.97 for Zymo, matching the claim of r2 > 0.97 when sample genomes or close relatives are in the database
- MetaMaps PotFrequency values are the final estimated frequencies after correcting for unmapped reads, and taxon counts should be consistent with the .EM.reads2Taxon assignments
- L1-distance and r2 are computed for every tool–dataset combination and written to composition_metrics.csv with matching per-tool profiles in composition_profiles.csv
- Reproduced CAMI compositional results match Fig. 2, Fig. 5, and Supplementary Data 3

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- Database-sample incongruity: complex samples comprising many non-characterized entities remain challenging, and detected database-sample mismatches may require visual analysis or automatic thresholding

## Evidence

- [abstract] Species-level accuracy and composition r2: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [intro] EM-based composition estimation: "each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [readme] WIMP composition output format: [evidence span withheld — anchor status: not_found]
- [readme] PotFrequency definition: [evidence span withheld — anchor status: not_found]
