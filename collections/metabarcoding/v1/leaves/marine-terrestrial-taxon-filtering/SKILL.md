---
name: marine-terrestrial-taxon-filtering
description: Use when a quality-controlled OTU/taxon table from multilocus eDNA amplicon
  sequencing needs terrestrial taxa removed and lower-rank assignments collapsed to
  Family or higher before downstream community or network analyses.
when_to_use_negative:
- Input is already an agglomerated, marine-only feature table with no terrestrial
  taxa present
- The analysis requires species-level taxonomic resolution, which the paper notes
  carries lower assignment confidence and is not what this agglomeration preserves
- The dataset is a single-locus amplicon study rather than the multilocus eDNA sequencing
  context this filtering was designed for
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: marinebon/eDNA_microbes_whales
  role: Source repository of the QC'd OTU/taxon outputs that are loaded, agglomerated
    to Family-or-higher rank, and filtered to marine taxa
  repo: github.com/marinebon/eDNA_microbes_whales
- name: pandas
  role: Table-processing environment for loading, agglomerating, filtering, and writing
    the marine taxa CSV
- name: phyloseq
  role: Alternative R environment for agglomerating OTU-level assignments to higher
    taxonomic rank
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:7d1725945c78a7b2e72c71404a3f400c194156842e67b2442611816b4cb2eee3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - marinebon/eDNA_microbes_whales
  - pandas
  - phyloseq
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# Marine-terrestrial taxon filtering and Family-level agglomeration

## Summary

Agglomerates QC'd multilocus eDNA OTU assignments to Family-or-higher taxonomic rank and filters out terrestrial taxa to isolate the marine subset of a biodiversity time-series, mirroring the Monterey Bay eDNA study's reduction of 663 taxonomic groups to 348 unique marine taxa.

## When to use

Use when a quality-controlled OTU/taxon table from multilocus eDNA amplicon sequencing needs terrestrial taxa removed and lower-rank assignments collapsed to Family or higher before downstream community or network analyses.

## When NOT to use

- Input is already an agglomerated, marine-only feature table with no terrestrial taxa present
- The analysis requires species-level taxonomic resolution, which the paper notes carries lower assignment confidence and is not what this agglomeration preserves
- The dataset is a single-locus amplicon study rather than the multilocus eDNA sequencing context this filtering was designed for

## Inputs

- QC'd OTU/taxon output tables from the eDNA_microbes_whales deposit (OTU-level taxonomic assignments across multiple loci)
- Taxonomic rank annotations per OTU/taxon row

## Outputs

- Annotated marine taxa table (marine_taxa_table.csv) with one row per unique marine taxon, Family-or-higher rank, and rank label
- Aggregate counts: total taxonomic groups (663), unique marine taxa (348), Family-level subset (274)

## How to apply

Load the QC'd OTU/taxon outputs from the github.com/marinebon/eDNA_microbes_whales deposit (the 2015-2016 eighteen-month Monterey Bay seawater time-series) into a table-processing environment such as pandas or phyloseq. Agglomerate OTU-level assignments to Family-or-higher taxonomic rank, collapsing lower-rank groupings, since the paper's dataset encompasses 663 taxonomic groups at that rank. Annotate each row with its assigned taxonomic rank, then filter to remove terrestrial taxa and retain only marine taxa. Verify the aggregated counts against the reported totals: 663 taxonomic groups overall, 348 unique marine taxa after terrestrial removal, and 274 (78%) of those agglomerated at the Family level. Finally write the annotated marine taxa table to marine_taxa_table.csv.

## Related tools

- **marinebon/eDNA_microbes_whales** (Source repository of the QC'd OTU/taxon outputs that are loaded, agglomerated to Family-or-higher rank, and filtered to marine taxa) — github.com/marinebon/eDNA_microbes_whales
- **pandas** (Table-processing environment for loading, agglomerating, filtering, and writing the marine taxa CSV)
- **phyloseq** (Alternative R environment for agglomerating OTU-level assignments to higher taxonomic rank)

## Examples

```
import pandas as pd; df = pd.read_table('otu_taxa.tsv'); df[df.domain.isin(['Bacteria','Archaea','Eukaryota']) & (~df.habitat.eq('terrestrial'))].to_csv('marine_taxa_table.csv', index=False)
```

## Evaluation signals

- Total agglomerated taxonomic groups equal 663 (Family or higher rank), matching the reported dataset size
- After removing terrestrial taxa, exactly 348 unique marine taxa remain
- 274 of the 348 marine taxa (78%) are agglomerated at the Family level; the remainder are pooled at higher taxonomic levels
- Every row in the output table carries a non-empty taxonomic rank annotation at Family or higher
- Output file marine_taxa_table.csv contains zero terrestrial taxa

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; primer biases, inhibitors, and variable template abundances affect which taxa are detected
- Lower taxonomic assignments carry lower confidence, so Family-level agglomeration trades resolution for assignment confidence
- eDNA cannot identify life history stages and may include sequences from dead or decaying biological matter

## Evidence

- [abstract] 663 taxonomic groups at Family-or-higher rank in the resulting dataset: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [abstract] Multilocus eDNA sequencing of the 2015-2016 Monterey Bay time-series: "we used multilocus amplicon sequencing of eDNA to survey biodiversity from an eighteen-month (2015–2016) time-series of seawater samples from Monterey Bay, California"
- [results] 348 unique marine taxa remain after terrestrial removal, 274 (78%) at Family level: [evidence span withheld — anchor status: not_found]
- [readme] Repository purpose: sequencing data analysis, quality filtering, and bioinformatics: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
