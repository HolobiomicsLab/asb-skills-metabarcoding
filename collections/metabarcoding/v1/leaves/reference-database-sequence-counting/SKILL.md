---
name: reference-database-sequence-counting
description: Use when comparing classifier training sets by taxonomic breadth and
  database size, or when verifying a published claim like '>3x taxonomic coverage'
  or 'almost five times as many reference sequences' from raw taxonomy/FASTA files.
when_to_use_negative:
- The goal is to measure classification accuracy or error rates — use leave-one-out
  or 5-fold cross-validation with bootstrap support cutoffs instead of counting reference
  records
- The goal is to actually classify query CO1 sequences — run the RDP classifier with
  the trained rRNAClassifier.properties file rather than parsing the training files
- The comparison must be between two recent versions with documented summary statistics
  (e.g. v4 vs v5 in the README already reports sequence/taxa/species deltas), so recounting
  from scratch adds little
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: RDP classifier
  role: Classifier whose training sets (taxonomy + FASTA reference files) are the
    objects being counted and compared
  repo: https://github.com/terrimporter/CO1Classifier
- name: Perl
  role: Used with BioPerl modules and the Ebot script to assemble the CO1 reference
    sequence sets (Arthropoda, Chordata, outgroup taxa) whose counts are being tallied
- name: pandas
  role: Python library for parsing the taxonomy files of both training sets to count
    unique taxa at the genus rank
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:687ecfc9aba84666ae5d062a6c8291d751e4fac2a3de541fc2d291e6a3f7dd42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - Perl
  - RDP classifier
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# reference-database-sequence-counting

## Summary

Quantifies taxonomic coverage and reference sequence counts of CO1 classifier training sets by parsing their taxonomy and FASTA files, enabling head-to-head verification of coverage claims (e.g. CO1 Eukaryote v1 vs the March 2013 Insecta Genbank-Genus set from Porter et al. 2014).

## When to use

Use when comparing classifier training sets by taxonomic breadth and database size, or when verifying a published claim like '>3x taxonomic coverage' or 'almost five times as many reference sequences' from raw taxonomy/FASTA files.

## When NOT to use

- The goal is to measure classification accuracy or error rates — use leave-one-out or 5-fold cross-validation with bootstrap support cutoffs instead of counting reference records
- The goal is to actually classify query CO1 sequences — run the RDP classifier with the trained rRNAClassifier.properties file rather than parsing the training files
- The comparison must be between two recent versions with documented summary statistics (e.g. v4 vs v5 in the README already reports sequence/taxa/species deltas), so recounting from scratch adds little

## Inputs

- Taxonomy file of the CO1 Eukaryote v1 training set (August 2016 GenBank snapshot, terrimporter/CO1Classifier v1.0-ref release)
- FASTA file of the CO1 Eukaryote v1 training set reference sequences
- Taxonomy file of the Insecta Genbank-Genus training set (March 2013, Porter et al. 2014 RDP classifier deposit)
- FASTA file of the Insecta Genbank-Genus training set reference sequences

## Outputs

- coverage_comparison.csv containing per-training-set genus counts, FASTA sequence counts, coverage ratio, sequence ratio, and verification verdicts against the >3x and ~5x claims

## How to apply

Download the reference ('-ref') release containing the original taxonomy and FASTA files for each training set being compared, e.g. the CO1 Eukaryote v1 set from terrimporter/CO1Classifier (v1.0-ref, August 2016 GenBank snapshot) and the Insecta Genbank-Genus set from Porter et al. (2014). Parse each taxonomy file with pandas to count unique taxa at the genus rank, noting that the CO1 Eukaryote v1 set includes outgroup taxa while the Insecta set covers class Insecta only — count coverage consistently with the claim being tested (including outgroups). Parse each FASTA file by counting '>' header lines to obtain reference sequence totals. Compute the coverage ratio and sequence ratio between sets and check them against the reported thresholds: the taxonomic coverage ratio must exceed 3x and the reference sequence ratio must be approximately 5x. Write counts, ratios, and per-claim verdicts to a CSV (e.g. coverage_comparison.csv).

## Related tools

- **RDP classifier** (Classifier whose training sets (taxonomy + FASTA reference files) are the objects being counted and compared) — https://github.com/terrimporter/CO1Classifier
- **Perl** (Used with BioPerl modules and the Ebot script to assemble the CO1 reference sequence sets (Arthropoda, Chordata, outgroup taxa) whose counts are being tallied)
- **pandas** (Python library for parsing the taxonomy files of both training sets to count unique taxa at the genus rank)

## Examples

```
python -c "import pandas as pd; tax=pd.read_csv('CO1v1_taxonomy.tsv',sep='\t'); n=[l for l in open('CO1v1.fasta') if l.startswith('>')]; print(tax['genus'].nunique(), len(n))"
```

## Evaluation signals

- Computed taxonomic coverage ratio between the CO1 Eukaryote v1 and 2013 Insecta genus counts exceeds 3x (coverage counted including outgroup taxa)
- Computed reference sequence ratio is approximately 5x, matching the abstract's 'almost five times as many reference sequences'
- FASTA '>'-header count per training set equals the number of reference sequences reported in the corresponding release notes or Table 1
- Unique genus counts derived from the taxonomy files are internally consistent between the taxonomy and FASTA files (each reference sequence maps to a taxon)
- coverage_comparison.csv contains explicit verdicts (pass/fail) for both claimed ratios rather than raw counts alone

## Limitations

- Coverage counts include outgroup taxa, so the >3x figure partially reflects broadening beyond class Insecta rather than Insecta-only gains
- Counts measure database representation only; GenBank CO1 coverage remains incomplete (only 22% of extant Insecta genera had full-length species-rank CO1 barcodes), so high counts do not imply exhaustive species coverage
- Counts are snapshot-specific (August 2016 vs March 2013); later releases (v3, v4, v5, v5.1.0) use different mining dates and filters (500bp+, no ambiguities, contaminant screening), so ratios are not transferable across versions
- False negatives remain possible at classification time even with a larger database when congeneric sequences are missing

## Evidence

- [abstract] Current classifier has more than three times the taxonomic coverage and almost five times as many reference sequences: "the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences"
- [methods] Three CO1 reference sequence sets assembled with Perl/BioPerl: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa"
- [results] Training sets usable directly with the RDP classifier: "Training sets can be used directly with the RDP classifier and no further training is needed from the user."
- [readme] README: reference releases contain original taxonomy and FASTA files: "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file)"
