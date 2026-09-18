---
name: metabarcoding-deposit-data-loading
description: Use when a QC'd multilocus eDNA OTU/taxon table from a published deposit
  must be loaded, agglomerated to Family-or-higher rank, and filtered to marine taxa
  for reproducibility checks.
when_to_use_negative:
- Input is raw FASTQ/amplicon reads that still require quality filtering and bioinformatics
  processing rather than an already-QC'd deposit table
- The analysis goal is species-level taxonomic assignment or biodiversity tracking,
  which the article notes carries lower confidence than Family-or-higher agglomeration
- The goal is community network or interaction analysis of taxa rather than tabulating
  and filtering taxonomic groups
tools:
- name: marinebon/eDNA_microbes_whales
  role: Deposit hosting the QC'd sequencing data analysis outputs that are loaded,
    agglomerated, and filtered by this skill
  repo: https://github.com/marinebon/eDNA_microbes_whales
- name: pandas / phyloseq
  role: Table-processing environments named in the workflow for loading and agglomerating
    the OTU/taxon table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:cf6f616a8f5f5fdb97d1b1d4caa35210665b6b5d53b4b1d540b885ea0d0a6ae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - marinebon/eDNA_microbes_whales
  - pandas / phyloseq
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 4
---

# metabarcoding-deposit-data-loading

## Summary

Loads the quality-controlled multilocus eDNA OTU/taxon outputs from the marinebon/eDNA_microbes_whales deposit, agglomerates assignments to Family-or-higher rank, and filters to marine taxa so reproducible biodiversity counts (e.g. 663 taxonomic groups, 348 marine taxa) can be verified against the published Monterey Bay 2015-2016 time-series results.

## When to use

Use when a QC'd multilocus eDNA OTU/taxon table from a published deposit must be loaded, agglomerated to Family-or-higher rank, and filtered to marine taxa for reproducibility checks.

## When NOT to use

- Input is raw FASTQ/amplicon reads that still require quality filtering and bioinformatics processing rather than an already-QC'd deposit table
- The analysis goal is species-level taxonomic assignment or biodiversity tracking, which the article notes carries lower confidence than Family-or-higher agglomeration
- The goal is community network or interaction analysis of taxa rather than tabulating and filtering taxonomic groups

## Inputs

- QC'd OTU/taxon table files from the github.com/marinebon/eDNA_microbes_whales deposit (multilocus eDNA sequencing outputs from the 2015-2016 Monterey Bay seawater time-series)
- Taxonomy assignment table mapping OTUs to taxonomic lineages and ranks

## Outputs

- Annotated marine taxa table written to marine_taxa_table.csv, with one row per unique marine taxon and its agglomeration rank (Family or higher)
- Verified count summary: 663 total taxonomic groups, 348 unique marine taxa, 274 (78%) agglomerated at Family level

## How to apply

Load the QC'd OTU/taxon outputs from github.com/marinebon/eDNA_microbes_whales into a table-processing environment such as pandas (Python) or phyloseq (R). Agglomerate OTU-level assignments to Family-or-higher taxonomic rank, collapsing lower-rank groupings, mirroring the paper's reported '663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals'. Filter the agglomerated table to remove terrestrial taxa, retaining only marine taxa, and annotate each row with the taxonomic rank at which it was agglomerated. Verify counts against the published totals: 663 taxonomic groups overall, 348 unique marine taxa after removing terrestrial taxa, of which 274 (78%) were agglomerated at Family level. Write the annotated marine taxa table to marine_taxa_table.csv.

## Related tools

- **marinebon/eDNA_microbes_whales** (Deposit hosting the QC'd sequencing data analysis outputs that are loaded, agglomerated, and filtered by this skill) — https://github.com/marinebon/eDNA_microbes_whales
- **pandas / phyloseq** (Table-processing environments named in the workflow for loading and agglomerating the OTU/taxon table)

## Examples

```
import pandas as pd; taxa = pd.read_csv('eDNA_microbes_whales/otu_taxa_table.csv'); taxa.to_csv('marine_taxa_table.csv', index=False)
```

## Evaluation signals

- Agglomerated table contains 663 taxonomic groups before terrestrial filtering, matching the published total
- After removing terrestrial taxa the marine table contains 348 unique taxa
- 274 of the 348 marine taxa (78%) are annotated at Family level, with the remainder annotated at higher rank
- No terrestrial taxa remain in the output table and every row carries an explicit taxonomic rank annotation
- Output file marine_taxa_table.csv exists and its row content matches the deposit's QC'd taxon assignments

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; detected counts are affected by primer biases, inhibitors, and variable template DNA abundances
- Species-level taxonomic assignment has lower confidence, which is why taxa are agglomerated at Family-or-higher rank
- The repository README is minimal and provides no script-level documentation, so loading conventions must be inferred from the deposit contents

## Evidence

- [abstract] The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals.: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [abstract] we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
