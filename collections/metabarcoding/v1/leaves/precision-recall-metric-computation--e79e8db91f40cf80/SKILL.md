---
name: precision-recall-metric-computation
description: Use when evaluating a metagenomic read classifier (e.g., MetaMaps, Kraken,
  Centrifuge) on datasets with known ground truth, to quantify strain- and species-level
  assignment accuracy.
when_to_use_negative:
- Samples contain a high proportion of short reads, which MetaMaps' minimizer-based
  approximate mapping cannot reliably assign (reads below the minimum length remain
  unassigned)
- The goal is de novo or novelty detection on samples with many non-characterized
  entities rather than accuracy benchmarking against known ground truth
- The dataset has no ground truth, so precision/recall cannot be computed
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: MetaMaps
  role: Long-read metagenomic classifier whose strain/species-level precision and
    recall are computed; assignments taken from classification_results.EM.reads2Taxon
  repo: https://github.com/DiltheyLab/MetaMaps
- name: Kraken
  role: Baseline k-mer-based classifier benchmarked on the same read sets for comparison
    of precision/recall
- name: Kraken 2
  role: Baseline k-mer-based classifier benchmarked on the same read sets for comparison
    of precision/recall
- name: MEGAN
  role: Baseline (LAST+MEGAN-LR) alignment-based classifier benchmarked on the same
    read sets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:c4f7ecb9f56fb5ed446c3b6749ba8964334f3649fe2701a85ef3fa8f76e5d599
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Kraken
  - Kraken 2
  - MEGAN
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# precision-recall-metric-computation

## Summary

Computes species- and strain-level precision, recall (at read and base level), and precision2 for metagenomic classifiers by comparing their taxonomic assignments against known ground truth, as done to benchmark MetaMaps against Centrifuge, Kraken, Kraken 2, and LAST+MEGAN-LR.

## When to use

Use when evaluating a metagenomic read classifier (e.g., MetaMaps, Kraken, Centrifuge) on datasets with known ground truth, to quantify strain- and species-level assignment accuracy.

## When NOT to use

- Samples contain a high proportion of short reads, which MetaMaps' minimizer-based approximate mapping cannot reliably assign (reads below the minimum length remain unassigned)
- The goal is de novo or novelty detection on samples with many non-characterized entities rather than accuracy benchmarking against known ground truth
- The dataset has no ground truth, so precision/recall cannot be computed

## Inputs

- Per-read taxonomic assignment files (e.g., classification_results.EM.reads2Taxon from MetaMaps)
- Ground-truth taxonomic labels for simulated (i100, p25) and real (CAMI, HMP7, Zymo) long-read metagenomic datasets
- Classification database taxonomy (NCBI taxonomy plus MetaMaps extended taxonomy for 'x' taxon IDs)

## Outputs

- classification_metrics.csv with species-level precision, recall (read and base level), precision2, and strain-level precision/recall per classifier per dataset
- Strain-resolution estimates (e.g., 94%/89% for i100/p25 simulations; 65%/61% for HMP7)

## How to apply

Run each classifier (MetaMaps, Kraken, Kraken 2, Centrifuge, LAST+MEGAN-LR, Bracken) on the same read sets (simulated i100 and p25, and real CAMI, HMP7, Zymo) against a common reference database such as the >12,000-genome miniSeq+H/RefSeq database. For every classifier and dataset, compare per-read taxonomic assignments to the known ground truth and compute species-level precision, recall at both read and base level, and precision2. Additionally compute strain-resolution metrics at the 'definedGenomes' level, where MetaMaps performs its EM estimation. Compile all metrics into a single classification_metrics.csv and cross-check the values against published figures and supplementary data tables (Figs. 1 and 4, Supplementary Data 1-2).

## Related tools

- **MetaMaps** (Long-read metagenomic classifier whose strain/species-level precision and recall are computed; assignments taken from classification_results.EM.reads2Taxon) — https://github.com/DiltheyLab/MetaMaps
- **Kraken** (Baseline k-mer-based classifier benchmarked on the same read sets for comparison of precision/recall)
- **Kraken 2** (Baseline k-mer-based classifier benchmarked on the same read sets for comparison of precision/recall)
- **MEGAN** (Baseline (LAST+MEGAN-LR) alignment-based classifier benchmarked on the same read sets)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Species-level recall and precision for MetaMaps on simulated data reach ≥99%, with strain-level values of 94% (i100) and 89% (p25)
- MetaMaps outperforms Centrifuge by >30% at the strain level on simulated experiments
- On real HMP7 data, strain-level precision/recall of ~65%/61% is reproduced
- classification_metrics.csv values match those reported in Figs. 1 and 4 and Supplementary Data 1-2
- Sample composition estimates show r2 > 0.97 when sample genomes or close relatives are in the database

## Limitations

- Database-sample incongruity (sample taxa absent or divergent from the reference database) depresses precision and must be detected via coverage/identity inspection, as shown in the HMP7 and CAMI experiments
- Reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Complex samples with many non-characterized entities remain challenging, so metrics assume ground truth within the reference database

## Evidence

- [results] On the simulated i100 and p25 experiments, MetaMaps achieves strain-level recall and precision of 94% and 89%: [evidence span withheld — anchor status: not_found]
- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [readme] The level 'definedGenomes' represents strain-level resolution: [evidence span withheld — anchor status: not_found]
- [discussion] As we have shown in the HMP7 and CAMI experiments, visual analysis or automatic thresholding can contribute to detecting and dealing with database-sample mismatches: "As we have shown in the HMP7 and CAMI experiments, visual analysis or automatic thresholding can contribute to detecting and dealing with database-sample mismatches."
