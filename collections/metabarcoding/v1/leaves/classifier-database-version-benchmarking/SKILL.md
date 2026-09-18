---
name: classifier-database-version-benchmarking
description: Use when comparing two releases of a classifier training set (e.g. CO1
  Eukaryote v1 vs the 2013 Insecta Genbank-Genus set) to quantify taxonomic coverage
  and reference-sequence growth against published claims.
when_to_use_negative:
- The task is to run the classifier itself or assess accuracy (leave-one-out/cross-validation
  bootstrap cutoffs) rather than to compare training set composition across versions
- Only one training set version is available and no prior-version baseline can be
  obtained, making ratio computation impossible
- The comparison targets species- or BIN-level counts across releases (e.g. v4 vs
  v5.1.0 counts in the README) rather than the v1-vs-2013 genus-rank benchmark this
  workflow encodes
tools:
- name: RDP classifier
  role: The classifier whose training sets are being benchmarked; the 2013 Insecta
    Genbank-Genus training set is obtained from the Porter et al. (2014) RDP classifier
    deposit
- name: Perl
  role: Used in the source workflow context for assembling and handling the CO1 reference
    sequence sets and BioPerl modules (with Ebot) that produced the training sets
    under comparison
- name: CO1Classifier releases (terrimporter)
  role: Source of the CO1 Eukaryote v1 training set taxonomy and FASTA files used
    in the comparison
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:10a0c89b205c16a72c6ec7d69260307b398b4a379251d712edb1454036914f56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - CO1Classifier releases (terrimporter)
  - Perl
  - RDP classifier
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classifier-database-version-benchmarking

## Summary

Quantitatively benchmarks two versions of a taxonomic classifier training set by parsing their taxonomy and FASTA files to count unique taxa (e.g. at genus rank) and reference sequences, then computing ratios. Used here to verify that the CO1 Eukaryote v1 training set (August 2016) exceeds the 2013 Insecta Genbank-Genus classifier by >3x taxonomic coverage and ~5x reference sequences.

## When to use

Use when comparing two releases of a classifier training set (e.g. CO1 Eukaryote v1 vs the 2013 Insecta Genbank-Genus set) to quantify taxonomic coverage and reference-sequence growth against published claims.

## When NOT to use

- The task is to run the classifier itself or assess accuracy (leave-one-out/cross-validation bootstrap cutoffs) rather than to compare training set composition across versions
- Only one training set version is available and no prior-version baseline can be obtained, making ratio computation impossible
- The comparison targets species- or BIN-level counts across releases (e.g. v4 vs v5.1.0 counts in the README) rather than the v1-vs-2013 genus-rank benchmark this workflow encodes

## Inputs

- CO1 Eukaryote v1 training set taxonomy file (from terrimporter/CO1Classifier release v1.0-ref)
- CO1 Eukaryote v1 training set FASTA file (from terrimporter/CO1Classifier release v1.0-ref)
- Insecta Genbank-Genus training set (taxonomy + FASTA) from the Porter et al. (2014) RDP classifier deposit

## Outputs

- coverage_comparison.csv containing unique taxon counts at genus rank, FASTA sequence counts per training set, taxonomic coverage ratio, reference sequence ratio, and verification verdicts against the >3x / ~5x claims

## How to apply

Download both training sets: the CO1 Eukaryote v1 taxonomy and FASTA files from the terrimporter/CO1Classifier GitHub release (v1.0-ref) and the Insecta Genbank-Genus training set from the Porter et al. (2014) RDP classifier deposit. Parse each taxonomy file with pandas/Python to count unique taxa at the genus rank, keeping in mind that the CO1 Eukaryote v1 set includes outgroup taxa while the Insecta set covers class Insecta only — this scope difference must be reported alongside the ratio. Count reference sequences in each FASTA file, then compute the taxonomic coverage ratio and the reference sequence ratio. Validate against the paper's claim: coverage ratio must exceed 3x and the sequence ratio must be approximately 5x ('almost five times as many reference sequences'). Write the counts, ratios, and verification verdicts to coverage_comparison.csv.

## Related tools

- **RDP classifier** (The classifier whose training sets are being benchmarked; the 2013 Insecta Genbank-Genus training set is obtained from the Porter et al. (2014) RDP classifier deposit)
- **Perl** (Used in the source workflow context for assembling and handling the CO1 reference sequence sets and BioPerl modules (with Ebot) that produced the training sets under comparison)
- **CO1Classifier releases (terrimporter)** (Source of the CO1 Eukaryote v1 training set taxonomy and FASTA files used in the comparison) — https://github.com/terrimporter/CO1Classifier

## Examples

```
wget https://github.com/terrimporter/CO1Classifier/releases/download/RDP-COI-v5.1.0/RDP_COIv5.1.0.zip && unzip RDP_COIv5.1.0.zip
```

## Evaluation signals

- Computed taxonomic coverage ratio (CO1 Eukaryote v1 vs 2013 Insecta) must exceed 3x
- Computed reference sequence ratio must be approximately 5x ('almost five times as many')
- Genus-rank counts must be derived by parsing taxonomy files, and sequence counts by parsing FASTA files — the two counting methods must be reported separately
- The report must note that the v1 coverage figure includes outgroup taxa whereas the 2013 Insecta set covers class Insecta only
- All counts, ratios, and pass/fail verdicts must appear as rows in coverage_comparison.csv

## Limitations

- The >3x coverage claim includes outgroup taxa in the CO1 Eukaryote v1 set, so the ratio is not a like-for-like Insecta-only comparison
- The 2013 baseline is the Insecta Genbank-Genus classifier only, so the benchmark does not generalize to other prior classifiers or taxonomic scopes
- Newer CO1Classifier releases (e.g. v3-v5.1.0) use different filtering (500bp+, no nucleotide ambiguities, BIN consolidation, bacterial outgroups), so counts and cutoffs from the README do not correspond to the 2016 v1 benchmark numbers

## Evidence

- [abstract] Current classifier has more than three times the taxonomic coverage and almost five times as many reference sequences as the previous Insecta classifier: "the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences"
- [methods] Training set source location: "These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier"
- [results] Training sets used directly with RDP classifier: "Training sets can be used directly with the RDP classifier and no further training is needed from the user."
- [readme] README release download instructions for training sets: "The latest release can be downloaded from https://github.com/terrimporter/CO1Classifier/releases ."
- [readme] README example of per-release sequence and taxon counts used in version benchmarking: "This version is based on 2,216,547 COI sequences from 236,247 taxa including 185,389 species/BINs."
