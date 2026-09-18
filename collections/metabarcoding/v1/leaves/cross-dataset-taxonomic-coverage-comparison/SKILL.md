---
name: cross-dataset-taxonomic-coverage-comparison
description: Use when a study claims a new reference/training set improves on a predecessor,
  and the improvement must be verified by comparing taxonomic breadth (unique genera/taxa)
  and reference sequence counts parsed from the sets' taxonomy and FASTA files.
when_to_use_negative:
- The goal is to actually classify query CO1 sequences with the RDP classifier rather
  than benchmark training set composition
- The comparison targets classification accuracy metrics (e.g., leave-one-out correctness,
  false positive rates) rather than taxonomic coverage and sequence counts
- Only one training set is available, so no ratio-based comparison can be computed
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier
  role: The classifier whose training sets (CO1 Eukaryote v1 vs. Insecta Genbank-Genus)
    are being compared; training sets are formatted for the RDP classifier
  repo: https://github.com/terrimporter/CO1Classifier
- name: Perl
  role: Used with BioPerl modules (Ebot script) in the source study to assemble and
    process the CO1 reference sequence sets being compared
- name: pandas
  role: Parses the taxonomy files of both training sets to count unique taxa at the
    genus rank and compute coverage ratios
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:ee205156bce0c7eee1deda6b2edd9a6a477672dfa301001be52376d7cc340efc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - Perl
  - RDP classifier
  - pandas
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Cross-dataset taxonomic coverage comparison

## Summary

Quantitatively benchmarks two taxonomic classifier training sets (e.g., the 2016 CO1 Eukaryote v1 set vs. the 2013 Insecta Genbank-Genus set) by counting unique taxa at a chosen rank and total reference sequences, then computing coverage and sequence ratios to verify claims like 'more than three times the taxonomic coverage... and almost five times as many reference sequences'.

## When to use

Use when a study claims a new reference/training set improves on a predecessor, and the improvement must be verified by comparing taxonomic breadth (unique genera/taxa) and reference sequence counts parsed from the sets' taxonomy and FASTA files.

## When NOT to use

- The goal is to actually classify query CO1 sequences with the RDP classifier rather than benchmark training set composition
- The comparison targets classification accuracy metrics (e.g., leave-one-out correctness, false positive rates) rather than taxonomic coverage and sequence counts
- Only one training set is available, so no ratio-based comparison can be computed

## Inputs

- CO1 Eukaryote v1 training set taxonomy file (RDP-format, semicolon-delimited lineages) from terrimporter/CO1Classifier v1.0-ref release
- CO1 Eukaryote v1 training set FASTA reference sequence file
- Insecta Genbank-Genus training set (March 2013) taxonomy file from the Porter et al. (2014) RDP classifier deposit
- Insecta Genbank-Genus training set FASTA reference sequence file

## Outputs

- coverage_comparison.csv with per-set unique genus counts, reference sequence counts, coverage ratio, sequence ratio, and verification verdicts against the >3x coverage and ~5x sequence claims

## How to apply

Download both training sets' original training files (taxonomy and FASTA), e.g. the CO1 Eukaryote v1 set from the terrimporter/CO1Classifier GitHub 'v1.0-ref' release and the Insecta Genbank-Genus set from the Porter et al. (2014) RDP classifier deposit. Parse each taxonomy file (semicolon-delimited RDP lineage format) with pandas to count unique taxa at the genus rank, noting that the CO1 Eukaryote v1 set includes outgroup taxa while the Insecta set covers class Insecta only. Count reference sequences in each FASTA by parsing headers. Compute the coverage ratio and sequence ratio, and validate them against the paper's claims (>3x coverage including outgroups; almost 5x as many sequences); write counts, ratios, and verdicts to a CSV such as coverage_comparison.csv.

## Related tools

- **RDP classifier** (The classifier whose training sets (CO1 Eukaryote v1 vs. Insecta Genbank-Genus) are being compared; training sets are formatted for the RDP classifier) — https://github.com/terrimporter/CO1Classifier
- **Perl** (Used with BioPerl modules (Ebot script) in the source study to assemble and process the CO1 reference sequence sets being compared)
- **pandas** (Parses the taxonomy files of both training sets to count unique taxa at the genus rank and compute coverage ratios)

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('CO1v1_taxonomy.txt',sep=';',header=None); genera=t[0].str.rstrip(';').str.split(';').str[-1]; print(genera.nunique(), sum(1 for l in open('CO1v1.fasta') if l.startswith('>')));"
```

## Evaluation signals

- Computed taxonomic coverage ratio (unique genera, including outgroup taxa in the v1 set) exceeds 3x, matching the paper's claim
- Computed reference sequence ratio is approximately 5x, matching 'almost five times as many reference sequences'
- FASTA sequence counts match the number of records in the corresponding taxonomy files for each training set
- Outgroup taxa are present in the v1 set's genus counts (they must be included per the paper's claim) but not in the Insecta-only set
- coverage_comparison.csv contains counts, ratios, and a verification verdict for both claims

## Limitations

- The two training sets have different taxonomic scope (Eukaryote with outgroups vs. class Insecta only), so coverage comparisons depend on explicitly including outgroups in the newer set's counts
- The comparison reflects the August 2016 (v1) vs. March 2013 snapshots; later releases (e.g., v5.1.0 with 2,216,547 COI sequences from 236,247 taxa) would give very different ratios
- Genus-rank counts from RDP taxonomy files depend on consistent lineage parsing; annotation errors in database entries can distort taxon counts, which the paper mitigates by summarizing to genus or more inclusive ranks

## Evidence

- [abstract] Coverage claim grounding: "the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences"
- [methods] Reference set assembly grounding: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa"
- [readme] Training set file availability grounding: "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file)"
- [methods] Repository location grounding: "These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier"
