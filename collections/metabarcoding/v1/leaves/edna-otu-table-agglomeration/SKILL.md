---
name: edna-otu-table-agglomeration
description: Use when you have QC'd OTU/taxon outputs from a multilocus eDNA amplicon
  survey and need Family-or-higher-rank agglomerated taxon tables with terrestrial
  taxa removed to obtain the marine taxa subset.
when_to_use_negative:
- The input is already an agglomerated Family-or-higher taxon table rather than OTU-level
  assignments — re-agglomeration would be redundant
- The analysis requires species-level taxonomic resolution, which the authors note
  carries lower confidence in eDNA assignments
- The goal is to identify all taxa in the ecosystem, since the authors explicitly
  note that no method, including this one, identifies all taxa
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: marinebon/eDNA_microbes_whales
  role: Source repository holding the sequencing data analysis, quality filtering,
    and bioinformatics outputs (QC'd OTU/taxon tables) that this skill agglomerates,
    filters, and exports as marine_taxa_table.csv
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:d7155b139eb31040f7722cd76d33a5ccb50c8b22d7b2725e058ad288482b5f37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - github.com/marinebon/eDNA_microbes_whales
  - marinebon/eDNA_microbes_whales
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# eDNA OTU table agglomeration to Family-or-higher marine taxa

## Summary

Agglomerates QC'd multilocus eDNA OTU assignments from the Monterey Bay 2015–2016 time-series to Family-or-higher taxonomic rank, filters out terrestrial taxa, and verifies counts against the reported totals (663 taxonomic groups overall; 348 unique marine taxa, of which 274, or 78%, agglomerated at Family level).

## When to use

Use when you have QC'd OTU/taxon outputs from a multilocus eDNA amplicon survey and need Family-or-higher-rank agglomerated taxon tables with terrestrial taxa removed to obtain the marine taxa subset.

## When NOT to use

- The input is already an agglomerated Family-or-higher taxon table rather than OTU-level assignments — re-agglomeration would be redundant
- The analysis requires species-level taxonomic resolution, which the authors note carries lower confidence in eDNA assignments
- The goal is to identify all taxa in the ecosystem, since the authors explicitly note that no method, including this one, identifies all taxa

## Inputs

- QC'd OTU/taxon assignment table from the eDNA_microbes_whales deposit (OTU-level multilocus amplicon sequencing outputs)
- Taxonomy assignment column per OTU with rank information (Family or higher)
- Marine/terrestrial taxon annotation used to filter terrestrial taxa

## Outputs

- Agglomerated table of 663 taxonomic groups at Family-or-higher rank
- marine_taxa_table.csv: filtered marine taxa table (348 unique marine taxa) with each row annotated by taxonomic rank
- Verification counts: 274 (78%) marine taxa agglomerated at Family level, remainder pooled at higher level

## How to apply

Load the quality-controlled OTU/taxon outputs from the github.com/marinebon/eDNA_microbes_whales deposit into a table-processing environment such as pandas or phyloseq. Agglomerate OTU-level assignments to Family-or-higher taxonomic rank, collapsing lower-rank groupings so that each row is one taxonomic group annotated with its rank. Filter the agglomerated table to remove terrestrial taxa, retaining only marine taxa, and write the annotated result to marine_taxa_table.csv. Verify the aggregated counts against the reported totals: 663 taxonomic groups (at Family or higher taxonomic rank) overall, 348 unique marine taxa after terrestrial removal, and 274 (78%) of those agglomerated at the Family level. The rationale is that lower taxonomic assignments carry lower confidence, so Family-or-higher agglomeration keeps the biodiversity counts robust across microorganisms to mammals in the eighteen-month seawater time-series.

## Related tools

- **marinebon/eDNA_microbes_whales** (Source repository holding the sequencing data analysis, quality filtering, and bioinformatics outputs (QC'd OTU/taxon tables) that this skill agglomerates, filters, and exports as marine_taxa_table.csv) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; taxa = pd.read_csv('qc_otu_taxon_table.csv'); taxa[taxa['marine']==True].to_csv('marine_taxa_table.csv', index=False)
```

## Evaluation signals

- Total agglomerated taxonomic groups equal 663 (Family or higher rank), matching the reported dataset total
- After terrestrial removal, the marine subset contains exactly 348 unique taxa
- 274 of the 348 marine taxa (78%) are agglomerated at the Family level; the remainder are pooled at higher rank
- Every row of marine_taxa_table.csv carries a taxonomic-rank annotation and no terrestrial taxa remain
- Counts reproduce when the workflow is re-run from the QC'd OTU/taxon outputs in the eDNA_microbes_whales deposit

## Limitations

- Amplification of DNA is subject to primer biases, inhibitors, and variable template abundances, which affect the number of taxa detected; the agglomerated counts inherit these biases
- No method, including this one, identifies all taxa in an ecosystem
- The lower the taxonomy assignment, the lower the confidence — species-level resolution is not reliably supported, hence agglomeration to Family or higher
- eDNA does not allow identification of life history stages and could include sequences from dead or decaying biological matter, so taxon counts reflect detectable DNA rather than census abundance

## Evidence

- [abstract] 663 taxonomic groups at Family or higher: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [abstract] multilocus eDNA time-series input: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [readme] repository provides QC/bioinformatics outputs: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
