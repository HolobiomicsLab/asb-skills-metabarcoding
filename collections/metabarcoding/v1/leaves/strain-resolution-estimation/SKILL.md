---
name: strain-resolution-estimation
description: Use when long PacBio/Oxford Nanopore reads must be classified to strain-level
  resolution and sample composition estimated, especially when sample genomes or close
  relatives are in RefSeq (>12,000 genomes).
when_to_use_negative:
- Samples dominated by short reads — MetaMaps' minimizer-based approximate mapping
  leaves reads below the minimum length unassigned
- Complex samples with many uncharacterized entities, where database-sample incongruity
  (novel strains/species absent from RefSeq) is the main concern
- Applications requiring streaming/real-time classification, which MetaMaps does not
  support
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: MetaMaps
  role: 'Primary tool: two-stage approximate mapping + probabilistic scoring + EM
    composition estimation for strain-level read assignment'
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer/minhash-based approximate mapping algorithm used by MetaMaps
  repo: https://github.com/marbl/MashMap
- name: Kraken 2
  role: Baseline k-mer classifier for benchmarking strain/species precision and recall
    on the same read sets and database
- name: Centrifuge
  role: Baseline classifier, outperformed by >30% at strain level on simulated i100/p25
    sets
- name: MEGAN
  role: Baseline alignment-based classifier (LAST+MEGAN-LR) used for comparison of
    read-level assignments
- name: LomanLab mockcommunity (Zymo)
  role: Real nanopore GridION/PromethION mock community read sets for evaluating classification
    accuracy
  repo: LomanLab/mockcommunity
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:b3460df0d2fbe50f57e8088b02d39a2fca2a9d1f245aadb20e205b1118b1b0ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Centrifuge
  - Kraken
  - Kraken 2
  - LomanLab mockcommunity (Zymo)
  - MEGAN
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# strain-resolution-estimation

## Summary

Assign long-read metagenomic reads at strain level and estimate sample composition using MetaMaps' two-stage procedure (minimizer-based approximate mapping via MashMap, probabilistic mapping-quality scoring, and EM-based composition estimation) against a comprehensive RefSeq database, benchmarked against k-mer classifiers such as Kraken, Kraken 2, and Centrifuge.

## When to use

Use when long PacBio/Oxford Nanopore reads must be classified to strain-level resolution and sample composition estimated, especially when sample genomes or close relatives are in RefSeq (>12,000 genomes).

## When NOT to use

- Samples dominated by short reads — MetaMaps' minimizer-based approximate mapping leaves reads below the minimum length unassigned
- Complex samples with many uncharacterized entities, where database-sample incongruity (novel strains/species absent from RefSeq) is the main concern
- Applications requiring streaming/real-time classification, which MetaMaps does not support

## Inputs

- Long-read FASTQ files (PacBio/ONT), e.g. simulated i100/p25 read sets, CAMI, HMP7, Zymo mock community
- MetaMaps classification database (e.g. miniSeq+H: >12,000 microbial genomes + human reference) with taxonomy sub-directory
- Ground-truth taxonomic assignments for benchmarking precision/recall

## Outputs

- classification_results.EM.WIMP: sample composition at taxonomic levels incl. strain ('definedGenomes') with Absolute, EMFrequency, PotFrequency columns
- classification_results.EM.reads2Taxon: per-read taxon assignments (incl. 'x' internal taxon IDs for strain disambiguation)
- classification_results.EM: per-read approximate mapping locations with estimated alignment identity and mapping quality
- classification_metrics.csv: species/strain precision, recall (read and base level), precision2 per classifier per dataset

## How to apply

Build MetaMaps from source with ./bootstrap.sh, ./configure --with-boost=/path/to/boost, and make metamaps, then run its two commands per dataset: mapDirectly against the miniSeq+H/RefSeq database (>12,000 genomes plus human), followed by classify, which scores each mapping location probabilistically and runs EM estimation of sample composition. Evaluate strain-level recall and precision against known ground truth, e.g. 94%/89% on simulated i100/p25 reads, ≥99% at species level, and 65%/61% strain metrics on the real HMP7 set. Compare baselines (Centrifuge, Kraken, Kraken 2, LAST+MEGAN-LR, Bracken) on the same read sets and database to verify the >30% strain-level advantage over Centrifuge; inspect the .EM.WIMP composition file (EMFrequency/PotFrequency) and .EM.reads2Taxon per-read assignments to confirm strain-level ('definedGenomes') output.

## Related tools

- **MetaMaps** (Primary tool: two-stage approximate mapping + probabilistic scoring + EM composition estimation for strain-level read assignment) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer/minhash-based approximate mapping algorithm used by MetaMaps) — https://github.com/marbl/MashMap
- **Kraken 2** (Baseline k-mer classifier for benchmarking strain/species precision and recall on the same read sets and database)
- **Centrifuge** (Baseline classifier, outperformed by >30% at strain level on simulated i100/p25 sets)
- **MEGAN** (Baseline alignment-based classifier (LAST+MEGAN-LR) used for comparison of read-level assignments)
- **LomanLab mockcommunity (Zymo)** (Real nanopore GridION/PromethION mock community read sets for evaluating classification accuracy) — LomanLab/mockcommunity

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Strain-level recall/precision reach ~94%/89% on simulated i100/p25 sets and 65%/61% on real HMP7 when assignments are compared to ground truth
- Species-level metrics ≥99% recall/precision and >94% species-level read-assignment accuracy
- Composition estimation r2 > 0.97 for sample composition when close relatives are in the database
- classification_results.EM.WIMP 'definedGenomes' level contains the EM-estimated strain-level frequencies (EMFrequency, PotFrequency)
- Mapping qualities in classification_results.EM sum to 1 across all mappings for each read

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Large 'contaminant' handling aside, no explicit genomic-divergence term in the statistical models limits explicit detection of novel strains/species; database-sample mismatches require visual coverage/identity inspection or thresholding
- Requires ~<16 GB RAM but memory target via --maxmemory is heuristic and actual usage fluctuates; no streaming-data support for clinical/real-time use

## Evidence

- [abstract] MetaMaps is a new method, specifically developed for long reads, capable of mapping a long-read metagenome to a comprehensive RefSeq database: "MetaMaps is a new method, specifically developed for long reads, capable of mapping a long-read metagenome to a comprehensive RefSeq database"
- [intro] Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm.: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [methods] ./bootstrap.sh ./configure --with-boost=/path/to/boost [--prefix=...] make metamaps: "./bootstrap.sh
./configure --with-boost=/path/to/boost [--prefix=...] 
make metamaps"
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] ./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results: [evidence span withheld — anchor status: not_found]
- [readme] The 'miniSeq+H' database is a good place to start. It contains >12000 microbial genomes and the human reference genome.: [evidence span withheld — anchor status: not_found]
