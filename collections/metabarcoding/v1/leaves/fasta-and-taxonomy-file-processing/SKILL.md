---
name: fasta-and-taxonomy-file-processing
description: Use when comparing or auditing RDP classifier training sets — e.g., quantifying
  how many genera and reference sequences a CO1 training set contains, or reproducing
  coverage comparisons between classifier versions.
when_to_use_negative:
- The goal is to classify query sequences rather than characterize a training set
  — use the trained RDP classifier files (rRNAClassifier.properties) with the classify
  command instead of re-parsing reference files
- The training sets being compared are not the specific releases under test (e.g.,
  mixing the latest v5.1.0 release, based on 2,216,547 COI sequences from 236,247
  taxa, with the 2013 Insecta set would not reproduce the 3x/5x comparison)
- The input is a trained RDP classifier binary/properties file with no original taxonomy/FASTA
  files available, since genus and sequence counts require the 'version-ref' source
  files
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RDP classifier
  role: The classifier whose training set files (taxonomy + FASTA) are parsed and
    compared; training sets can be used directly with it without further user training
  repo: https://github.com/terrimporter/CO1Classifier
- name: Perl
  role: Used with BioPerl modules and the Ebot script to assemble the three CO1 reference
    sequence sets (Arthropoda, Chordata, outgroup taxa) whose files are then processed
- name: BioPerl
  role: BioPerl modules used alongside Perl and Ebot to assemble CO1 reference sequence
    sets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:e90a6dd0e5f47f8a5985af8c30f11d9dee69e7427df74e8522ba54adb72cf03f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BioPerl
  - Perl
  - RDP classifier
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Fasta and taxonomy file processing

## Summary

Parses the paired taxonomy and FASTA reference files of RDP-classifier CO1 training sets (e.g., the CO1 Eukaryote v1 set from terrimporter/CO1Classifier and the 2013 Insecta Genbank-Genus set) to count unique taxa at genus rank and total reference sequences, enabling quantitative comparison of taxonomic coverage and training set size.

## When to use

Use when comparing or auditing RDP classifier training sets — e.g., quantifying how many genera and reference sequences a CO1 training set contains, or reproducing coverage comparisons between classifier versions.

## When NOT to use

- The goal is to classify query sequences rather than characterize a training set — use the trained RDP classifier files (rRNAClassifier.properties) with the classify command instead of re-parsing reference files
- The training sets being compared are not the specific releases under test (e.g., mixing the latest v5.1.0 release, based on 2,216,547 COI sequences from 236,247 taxa, with the 2013 Insecta set would not reproduce the 3x/5x comparison)
- The input is a trained RDP classifier binary/properties file with no original taxonomy/FASTA files available, since genus and sequence counts require the 'version-ref' source files

## Inputs

- RDP classifier taxonomy file (semicolon-delimited lineage records) from a CO1 training set release (e.g., v1.0-ref)
- Paired FASTA file of CO1 reference sequences for the same training set
- Comparison training set files (Insecta Genbank-Genus training set, March 2013, from the Porter et al. 2014 RDP classifier deposit)

## Outputs

- Per-set counts of unique taxa at the genus rank (noting outgroup inclusion)
- Per-set reference sequence counts from FASTA parsing
- Taxonomic coverage ratio and reference sequence ratio between training sets (e.g., written to coverage_comparison.csv) with verification verdicts against reported claims

## How to apply

Download the original training files (a taxonomy file and a FASTA file) from the 'version-ref' release of the terrimporter/CO1Classifier repository, making sure to pin the exact release (v1.0-ref for the 2016 CO1 Eukaryote v1 set) and the Porter et al. (2014) Insecta Genbank-Genus deposit for comparison. Parse the RDP-format taxonomy files with pandas/Python to count unique taxa at the genus rank, noting whether outgroup taxa are included (the CO1 Eukaryote v1 set covers Eukaryote taxa plus outgroups, while the 2013 Insecta set covers class Insecta only — the comparison must account for this asymmetry). Count reference sequences by parsing the FASTA files, then compute the genus-coverage ratio and reference-sequence ratio between sets. The paper reports the v1 classifier has 'more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences' than the previous Insecta classifier, so computed ratios should be checked against these thresholds.

## Related tools

- **RDP classifier** (The classifier whose training set files (taxonomy + FASTA) are parsed and compared; training sets can be used directly with it without further user training) — https://github.com/terrimporter/CO1Classifier
- **Perl** (Used with BioPerl modules and the Ebot script to assemble the three CO1 reference sequence sets (Arthropoda, Chordata, outgroup taxa) whose files are then processed)
- **BioPerl** (BioPerl modules used alongside Perl and Ebot to assemble CO1 reference sequence sets)

## Examples

```
python -c "import pandas as pd; tax = pd.read_csv('CO1v1_taxonomy.txt', sep=';', header=None); print(tax[tax[2]=='genus'][3].nunique())"
```

## Evaluation signals

- Computed taxonomic coverage ratio between the 2016 CO1 Eukaryote v1 set and the 2013 Insecta set exceeds 3x
- Computed reference sequence ratio is approximately 5x, matching 'almost five times as many reference sequences'
- Genus counts derived from the taxonomy file agree with counts from parsing the paired FASTA headers (no orphan records between files)
- Comparison explicitly notes that the v1 coverage figure includes outgroup taxa while the Insecta set covers class Insecta only
- Counts, ratios, and verification verdicts are recorded in a machine-readable output such as coverage_comparison.csv

## Limitations

- Coverage comparison conflates taxonomic breadth with outgroup inclusion — the 3x coverage claim includes outgroups, so a like-for-like Insecta-only comparison would be smaller
- Training set statistics are release-specific and drift rapidly: the latest README release (v5.1.0) is based on 2,216,547 COI sequences from 236,247 taxa, so counts from a pinned release (v1.0-ref) must not be generalized
- Only 22% of extant Insecta genera (16,285/72,618) are represented by full-length species-rank CO1 barcodes in GenBank, so counted coverage substantially underestimates true diversity

## Evidence

- [abstract] the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences: "the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences"
- [methods] These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier: "These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier"
- [methods] Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa"
- [readme] The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file): "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file) are available as 'version-ref'."
- [discussion] 22% of extant Insecta genera (16,285/72,618) are now represented by full length sequences identified to the species rank: "22% of extant Insecta genera (16,285/72,618) are now represented by full length sequences identified to the species rank"
