---
name: taxonomic-rank-annotation
description: Use when a QC'd eDNA OTU table contains taxonomic assignments at mixed
  ranks and you need Family-or-higher agglomeration, per-row rank annotation, and
  marine/terrestrial filtering before reporting taxon counts.
when_to_use_negative:
- Input is already agglomerated at Family or higher rank with rank annotations — re-agglomeration
  would double-collapse taxa
- The goal is species-level biodiversity reporting — the paper notes 'the lower the
  taxonomy assignment, the lower the confidence in that assignment becomes', so species-level
  claims need expanded reference databases rather than Family-level pooling
- The dataset is not a multilocus eDNA amplicon time-series and lacks lineage/provenance
  labels needed to remove terrestrial taxa
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: marinebon/eDNA_microbes_whales
  role: Source of the QC'd OTU/taxon outputs that are loaded, agglomerated, filtered
    to marine taxa, and exported as marine_taxa_table.csv
  repo: github.com/marinebon/eDNA_microbes_whales
- name: pandas
  role: Table-processing environment for loading the OTU table, agglomerating to Family-or-higher
    rank, filtering terrestrial taxa, and writing CSV output
- name: phyloseq
  role: Alternative R environment for taxonomic agglomeration of OTU-level assignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:1002fa1d80f1a2e244e32cf3c04e49fb1ceb7e9a6ca141689fe017b9d7517328
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - marinebon/eDNA_microbes_whales
  - pandas
  - phyloseq
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# taxonomic-rank-annotation

## Summary

Agglomerates QC'd OTU-level taxonomic assignments from multilocus eDNA amplicon sequencing to Family-or-higher rank, annotates each agglomerated row with its taxonomic rank, and separates marine from terrestrial taxa so biodiversity totals (663 taxonomic groups; 348 unique marine taxa, 274 at Family level) can be verified against the published time-series dataset.

## When to use

Use when a QC'd eDNA OTU table contains taxonomic assignments at mixed ranks and you need Family-or-higher agglomeration, per-row rank annotation, and marine/terrestrial filtering before reporting taxon counts.

## When NOT to use

- Input is already agglomerated at Family or higher rank with rank annotations — re-agglomeration would double-collapse taxa
- The goal is species-level biodiversity reporting — the paper notes 'the lower the taxonomy assignment, the lower the confidence in that assignment becomes', so species-level claims need expanded reference databases rather than Family-level pooling
- The dataset is not a multilocus eDNA amplicon time-series and lacks lineage/provenance labels needed to remove terrestrial taxa

## Inputs

- QC'd OTU/taxon table from the multilocus eDNA time-series deposit (github.com/marinebon/eDNA_microbes_whales) with OTU-level taxonomic assignments
- Taxonomy assignment strings per OTU including lineage ranks and terrestrial/marine provenance information

## Outputs

- marine_taxa_table.csv: agglomerated Family-or-higher taxa table, one row per taxonomic group, annotated with taxonomic rank and marine/terrestrial status
- Verified count summaries: total taxonomic groups (663), unique marine taxa (348), Family-level agglomerated subset (274, 78%)

## How to apply

Load the QC'd OTU/taxon outputs from the marinebon/eDNA_microbes_whales deposit into a table-processing environment such as pandas or phyloseq. Agglomerate OTU-level assignments to Family-or-higher taxonomic rank, collapsing lower-rank groupings, since the paper reports biodiversity at 'Family or higher taxonomic rank'. Filter the agglomerated table to remove terrestrial taxa, retaining only marine taxa, and annotate each row with the rank it was agglomerated to. Verify aggregate counts against reported totals: 663 taxonomic groups overall, 348 unique marine taxa after terrestrial removal, of which 274 (78%) were agglomerated at Family level and the remainder pooled at a higher taxonomic level. Write the rank-annotated marine taxa table out as marine_taxa_table.csv.

## Related tools

- **marinebon/eDNA_microbes_whales** (Source of the QC'd OTU/taxon outputs that are loaded, agglomerated, filtered to marine taxa, and exported as marine_taxa_table.csv) — github.com/marinebon/eDNA_microbes_whales
- **pandas** (Table-processing environment for loading the OTU table, agglomerating to Family-or-higher rank, filtering terrestrial taxa, and writing CSV output)
- **phyloseq** (Alternative R environment for taxonomic agglomeration of OTU-level assignments)

## Examples

```
import pandas as pd; otu = pd.read_csv('otu_taxa_qc.csv'); fam = otu.groupby(['kingdom','phylum','class','Family']).size().reset_index(); marine = fam[~fam.kingdom.isin(terrestrial)]; marine.to_csv('marine_taxa_table.csv')
```

## Evaluation signals

- Total distinct taxonomic groups at Family-or-higher rank equals 663, matching the reported dataset scope
- After terrestrial removal, exactly 348 unique marine taxa remain
- Of the 348 marine taxa, 274 (78%) are agglomerated at Family level and the remainder at a higher taxonomic level
- Every row in marine_taxa_table.csv carries a non-empty taxonomic-rank annotation and no terrestrial taxa are retained
- Output CSV parses cleanly and row count equals the marine taxa count (348)

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; primer biases, inhibitors, and variable template abundances affect which taxa are detected
- Species-level assignment has lower confidence — the lower the taxonomy assignment, the lower the confidence — motivating agglomeration at Family or higher rank
- eDNA cannot identify life-history stages and may include sequences from dead or decaying biological matter, so the marine taxa table reflects detections, not necessarily living community members

## Evidence

- [abstract] 663 taxonomic groups at Family-or-higher rank from multilocus eDNA: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [discussion] Species-level assignment confidence caveat: "Biodiversity fluctuations with time would ideally be observed on a species level, albeit the lower the taxonomy assignment, the lower the confidence in that assignment becomes."
- [readme] Repository purpose for eDNA sequencing data analysis: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
