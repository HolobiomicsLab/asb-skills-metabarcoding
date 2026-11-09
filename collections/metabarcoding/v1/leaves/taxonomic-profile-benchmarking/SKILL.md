---
name: taxonomic-profile-benchmarking
description: Use when you need to reproduce or compare strain- and species-level read-assignment
  accuracy (precision, recall, precision2) and composition-estimation accuracy (r2,
  L1 distance) of long-read metagenomic classifiers on mock or simulated communities
  with known ground truth.
when_to_use_negative:
- Samples dominated by short reads — the article notes that analysis of samples with
  a high proportion of short reads is not well addressed by MetaMaps because minimizer
  sets become too small
- Samples whose organisms are absent or poorly represented in the reference database
  — database-sample incongruity (yet-uncharacterized taxonomic entities) is an acknowledged
  unresolved concern
- Complex samples comprising many non-characterized entities, which the authors state
  remain challenging
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Primary long-read classifier and EM-based sample composition estimator being
    benchmarked
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Approximate mapping algorithm underlying MetaMaps' minimizer-based mapping
    stage
  repo: https://github.com/marbl/MashMap
- name: Kraken 2
  role: k-mer-based baseline classifier for precision/recall comparison
- name: Centrifuge
  role: Baseline classifier for per-read assignment comparison at strain and species
    levels
- name: Bracken
  role: Baseline whose per-read classifications are aggregated into composition profiles
- name: MEGAN
  role: LAST+MEGAN-LR baseline for LCA-style per-read assignment aggregation
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:9ce796cfc80346179c3281d2d2002c891f59cc939e69e4afecb23eda40b1b9c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Bracken
  - Centrifuge
  - Kraken
  - Kraken 2
  - LAST
  - MEGAN
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# taxonomic-profile-benchmarking

## Summary

Benchmark long-read metagenomic classifiers and sample-composition estimators (MetaMaps vs Centrifuge, Kraken/Kraken 2, Bracken, LAST+MEGAN-LR) on simulated (i100, p25) and real (HMP7, Zymo, CAMI) datasets against the >12,000-genome RefSeq miniSeq+H database, computing per-read precision/recall and Pearson r2/L1 distance between estimated and true compositions.

## When to use

Use when you need to reproduce or compare strain- and species-level read-assignment accuracy (precision, recall, precision2) and composition-estimation accuracy (r2, L1 distance) of long-read metagenomic classifiers on mock or simulated communities with known ground truth.

## When NOT to use

- Samples dominated by short reads — the article notes that analysis of samples with a high proportion of short reads is not well addressed by MetaMaps because minimizer sets become too small
- Samples whose organisms are absent or poorly represented in the reference database — database-sample incongruity (yet-uncharacterized taxonomic entities) is an acknowledged unresolved concern
- Complex samples comprising many non-characterized entities, which the authors state remain challenging

## Inputs

- Long-read FASTQ files from simulated (i100, p25) and real (CAMI, HMP7, Zymo) metagenomic datasets
- MetaMaps miniSeq+H classification database (RefSeq-derived, >12,000 genomes + human reference)
- Ground-truth taxonomic compositions for each mock/simulated community
- Per-read taxonomic assignment files from baseline classifiers (Centrifuge, Kraken, Kraken 2, Bracken, LAST+MEGAN-LR)

## Outputs

- classification_metrics.csv: species-level precision, recall (read and base level), precision2, and strain-resolution metrics per classifier per dataset
- composition_metrics.csv: Pearson r2 and L1-distance between estimated and true compositions per tool-dataset combination
- composition_profiles.csv: per-tool estimated taxonomic composition profiles
- MetaMaps outputs: .EM.WIMP (sample composition), .EM.reads2Taxon (per-read assignments), .EM (mappings with mapping qualities)

## How to apply

Build MetaMaps from source (./bootstrap.sh, ./configure --with-boost=/path/to/boost, make metamaps), download the miniSeq+H database (>12,000 microbial genomes plus the human reference), and run the two-step mapDirectly + classify procedure on each read set (FASTQ). Convert MetaMaps per-read assignments (classification_results.EM.reads2Taxon) and its EM-based sample composition (classification_results.EM.WIMP, using EMFrequency/PotFrequency) into taxon-level profiles; aggregate per-read classifications of Centrifuge, Bracken, Kraken, Kraken 2, and LAST+MEGAN-LR into comparable profiles. Match estimated compositions to ground truth at strain level for i100 and p25 and at species level for HMP7, Zymo, and CAMI, then compute Pearson r2 and L1 distance per tool-dataset combination. Validate against the reported values: strain-level recall/precision of 94%/89% (i100) and species-level r2 of 0.98 (HMP7) and 0.97 (Zymo), with strain-level r2 of 0.88 (i100) and 0.78 (p25) rising to >0.99 at higher taxonomic levels. Write metrics to composition_metrics.csv / classification_metrics.csv and profiles to composition_profiles.csv.

## Related tools

- **MetaMaps** (Primary long-read classifier and EM-based sample composition estimator being benchmarked) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Approximate mapping algorithm underlying MetaMaps' minimizer-based mapping stage) — https://github.com/marbl/MashMap
- **Kraken 2** (k-mer-based baseline classifier for precision/recall comparison)
- **Centrifuge** (Baseline classifier for per-read assignment comparison at strain and species levels)
- **Bracken** (Baseline whose per-read classifications are aggregated into composition profiles)
- **MEGAN** (LAST+MEGAN-LR baseline for LCA-style per-read assignment aggregation)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Strain-level recall/precision for MetaMaps on i100 should reach 94%/89% and species-level metrics >=99%; HMP7 strain-resolution ~65%/61%
- Strain-level Pearson r2 between estimated and true abundances should be ~0.88 (i100) and ~0.78 (p25), rising to >0.99 at genus and higher levels
- Species-level r2 should be ~0.98 (HMP7) and ~0.97 (Zymo), consistent with the abstract claim of r2 > 0.97 when sample genomes or close relatives are in the database
- MetaMaps .EM.WIMP columns EMFrequency and PotFrequency must sum consistently, with PotFrequency correcting for unmapped reads
- Mapping qualities of all mappings for one read must sum to 1 in classification_results.EM

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- No support for streaming data, limiting clinical/real-time sequencing applications
- No explicit term for genomic divergence in the statistical models, limiting explicit detection of novel strains and species
- Large contaminant genomes and database-sample mismatches (e.g., in HMP7 and CAMI) require visual analysis or automatic thresholding to detect

## Evidence

- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [intro] First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29.: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] Output columns: Absolute specifies the number of reads assigned (by their maximum likelihood mapping estimate) to the taxonomic entity; EMFrequency specifies the estimated frequency: [evidence span withheld — anchor status: not_found]
- [readme] The mapping qualities of all mappings for one read sum up to 1.: [evidence span withheld — anchor status: not_found]
