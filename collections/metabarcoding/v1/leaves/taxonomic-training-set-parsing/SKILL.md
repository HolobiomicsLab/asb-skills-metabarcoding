---
name: taxonomic-training-set-parsing
description: Use when you hold two or more RDP classifier training sets (a taxonomy
  file plus a FASTA file each) and need to verify claims about how many taxa and reference
  sequences each covers.
when_to_use_negative:
- You need to classify new query sequences with the RDP classifier rather than characterize
  training set contents — use the trained rRNAClassifier.properties files directly
  instead.
- Your training set is not formatted as an RDP-style taxonomy file plus FASTA pair
  (e.g. you only have BLAST or SINTAX formatted references, which the README distributes
  separately as of v5.0.0).
- You are comparing v1 and v2 CO1 training sets for species-rank assessment — the
  v2 species-trained set has different singleton proportions (33% vs 23% singleton
  genera) and requires rank-aware comparison, not simple genus counting.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier
  role: Defines the training set format (taxonomy file plus FASTA file) that this
    skill parses; training sets can be used directly with the RDP classifier with
    no further training.
  repo: https://github.com/terrimporter/CO1Classifier
- name: Perl
  role: Used with BioPerl modules in the original study to assemble the CO1 reference
    sequence sets that constitute the training sets being parsed.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:00303222c2a4a6d5e25a810ee5d4cf71c99a8e8be8314d2bcda803ad394721da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - Perl
  - RDP classifier
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# taxonomic-training-set-parsing

## Summary

Parses RDP classifier training sets (taxonomy and FASTA reference files) from the CO1Classifier releases to quantify taxonomic coverage (unique taxa at genus rank) and reference sequence counts, enabling direct comparison against older training sets such as the March 2013 Insecta Genbank-Genus classifier.

## When to use

Use when you hold two or more RDP classifier training sets (a taxonomy file plus a FASTA file each) and need to verify claims about how many taxa and reference sequences each covers.

## When NOT to use

- You need to classify new query sequences with the RDP classifier rather than characterize training set contents — use the trained rRNAClassifier.properties files directly instead.
- Your training set is not formatted as an RDP-style taxonomy file plus FASTA pair (e.g. you only have BLAST or SINTAX formatted references, which the README distributes separately as of v5.0.0).
- You are comparing v1 and v2 CO1 training sets for species-rank assessment — the v2 species-trained set has different singleton proportions (33% vs 23% singleton genera) and requires rank-aware comparison, not simple genus counting.

## Inputs

- RDP classifier training set taxonomy file (e.g. CO1 Eukaryote v1, from the v1.0-ref release)
- RDP classifier training set FASTA reference sequence file (e.g. CO1 Eukaryote v1)
- Comparison training set taxonomy and FASTA files (e.g. Insecta Genbank-Genus, March 2013, Porter et al. 2014)

## Outputs

- coverage_comparison.csv with per-set taxon counts (genus rank), reference sequence counts, coverage ratio, sequence ratio, and verification verdicts against the >3x coverage and ~5x sequence claims

## How to apply

Download the CO1 Eukaryote v1 training set (taxonomy and FASTA files) from the terrimporter/CO1Classifier GitHub release (v1.0-ref) and the comparison set — e.g. the March 2013 Insecta Genbank-Genus training set from Porter et al. (2014). Parse each taxonomy file with pandas/Python, splitting the RDP lineage string by semicolons to count unique taxa at the genus rank, remembering that the CO1 Eukaryote v1 set includes outgroup taxa while the Insecta set covers class Insecta only. Count reference sequences by parsing each FASTA file (count of '>' headers). Compute the coverage ratio and sequence ratio between the sets and check them against the paper's claims of more than three times the taxonomic coverage and almost five times as many reference sequences. Write the counts, ratios, and pass/fail verdicts to an output CSV such as coverage_comparison.csv.

## Related tools

- **RDP classifier** (Defines the training set format (taxonomy file plus FASTA file) that this skill parses; training sets can be used directly with the RDP classifier with no further training.) — https://github.com/terrimporter/CO1Classifier
- **Perl** (Used with BioPerl modules in the original study to assemble the CO1 reference sequence sets that constitute the training sets being parsed.)

## Examples

```
python -c "import pandas as pd; tax=pd.read_csv('CO1v1_taxonomy.txt',sep=';',header=None,names=['id','rank','taxa']); print(tax['taxa'].str.split(';').str[5].nunique())"
```

## Evaluation signals

- Computed taxonomic coverage ratio (CO1 Eukaryote v1 vs Insecta 2013) exceeds 3x, including outgroup taxa.
- Computed reference sequence ratio is approximately 5x, matching the claim of 'almost five times as many reference sequences'.
- FASTA record count equals the number of '>' headers parsed, and taxonomy genus counts are computed after splitting semicolon-delimited RDP lineages.
- coverage_comparison.csv contains counts, ratios, and explicit verification verdicts for both claims.
- The v1 set's taxon count includes outgroup taxa while the Insecta set is restricted to class Insecta — a discrepancy check should confirm outgroups are not double-counted as Insecta coverage.

## Limitations

- The CO1 Eukaryote v1 training set includes outgroup taxa, so raw genus counts overstate metazoan (Insecta-comparable) coverage unless outgroups are accounted for.
- Reference database coverage remains incomplete — only 22% of extant Insecta genera (16,285/72,618) have full-length species-rank CO1 barcodes in GenBank, so counts reflect database coverage, not true diversity.
- Training set contents are version-dependent; newer releases (v3 through v5.1.0) differ substantially in size (e.g. v5.1.0 is based on 2,216,547 COI sequences from 236,247 taxa), so comparisons must use the matching release tags such as v1.0-ref.

## Evidence

- [abstract] the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences: "the current classifier has more than three times the taxonomic coverage, including outgroups, and is based on almost five times as many reference sequences"
- [methods] using Perl with Bio Perl modules and the Ebot script: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa as described below using Perl with Bio Perl modules"
- [readme] The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file) are available as 'version-ref'.: "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file) are available as 'version-ref'."
- [results] Taxa for the training set were sampled to emphasize Arthropoda and Chordata since these were the best-represented eukaryote phyla in the Gen Bank nucleotide database.: "Taxa for the training set were sampled to emphasize Arthropoda and Chordata since these were the best-represented eukaryote phyla in the Gen Bank nucleotide database."
