---
name: biodiversity-count-verification
description: Use when you hold an OTU/taxon assignment table from a QC'd eDNA amplicon
  study and must reproduce or verify a reported taxonomic group count after agglomeration
  and marine-only filtering.
when_to_use_negative:
- Input is raw sequencing data (e.g. FASTQ) that has not yet been quality-filtered
  and taxonomically assigned to OTUs — the skill operates downstream of bioinformatic
  QC.
- The analysis goal is species-level resolution of communities — the article notes
  the lower the taxonomy assignment, the lower the confidence, so species-level biodiversity
  tracking is not what this verification supports.
- The input table is already agglomerated to Family-or-higher rank — re-agglomeration
  would be redundant; only the terrestrial-filtering and count-verification steps
  would apply.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: github.com/marinebon/eDNA_microbes_whales
  role: Source deposit of QC'd sequencing data, quality filtering, and bioinformatics
    outputs; its OTU/taxon tables are the inputs agglomerated and marine-filtered
    by this skill.
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:3b3bac9501b4b109450b3410b6f559034af21b3cfb726d8e465e173a812cb90f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - github.com/marinebon/eDNA_microbes_whales
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# biodiversity-count-verification

## Summary

Reproduces the curated biodiversity inventory from the Djurhuus et al. 2020 Monterey Bay eDNA time-series by agglomerating QC'd multilocus OTU assignments to Family-or-higher rank and filtering to marine taxa, yielding the reported totals of 663 taxonomic groups, 348 unique marine taxa, and 274 (78%) agglomerated at Family level.

## When to use

Use when you hold an OTU/taxon assignment table from a QC'd eDNA amplicon study and must reproduce or verify a reported taxonomic group count after agglomeration and marine-only filtering.

## When NOT to use

- Input is raw sequencing data (e.g. FASTQ) that has not yet been quality-filtered and taxonomically assigned to OTUs — the skill operates downstream of bioinformatic QC.
- The analysis goal is species-level resolution of communities — the article notes the lower the taxonomy assignment, the lower the confidence, so species-level biodiversity tracking is not what this verification supports.
- The input table is already agglomerated to Family-or-higher rank — re-agglomeration would be redundant; only the terrestrial-filtering and count-verification steps would apply.

## Inputs

- QC'd OTU/taxon assignment table(s) from the github.com/marinebon/eDNA_microbes_whales deposit (multilocus eDNA amplicon time-series, 18-month Monterey Bay seawater samples)
- Taxonomy rank annotation per OTU (Family or higher assignment labels)

## Outputs

- marine_taxa_table.csv — marine taxa rows annotated with agglomerated taxonomic rank
- Verified count tallies: 663 total taxonomic groups, 348 unique marine taxa, 274 (78%) at Family level

## How to apply

Load the quality-controlled OTU/taxon outputs from the github.com/marinebon/eDNA_microbes_whales deposit into a table-processing environment such as pandas or phyloseq. Agglomerate OTU-level assignments to Family-or-higher taxonomic rank, collapsing lower-rank groupings so each row is a taxonomic group, not an OTU. Filter the agglomerated table to remove terrestrial taxa, retaining only marine taxa, and annotate each row with the taxonomic rank at which it was pooled (Family vs. higher). Verify counts against the deposit's reported totals: 663 taxonomic groups (at Family or higher rank) overall, 348 unique marine taxa after removing terrestrial taxa, and 274 of those (78%) agglomerated at the Family level with the rest pooled at a higher level. Write the annotated marine taxa table to marine_taxa_table.csv.

## Related tools

- **github.com/marinebon/eDNA_microbes_whales** (Source deposit of QC'd sequencing data, quality filtering, and bioinformatics outputs; its OTU/taxon tables are the inputs agglomerated and marine-filtered by this skill.) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
import pandas as pd; df = pd.read_csv('marine_taxa_table.csv'); assert len(df) == 348
```

## Evaluation signals

- Pre-filter row count equals 663 taxonomic groups (at Family or higher taxonomic rank).
- Post-filter row count equals 348 unique marine taxa after terrestrial taxa are removed.
- Of the 348 marine taxa, exactly 274 (78%) are agglomerated at the Family level and the remainder are pooled at a higher taxonomic level.
- Every retained row is annotated with its taxonomic rank and contains no terrestrial taxa.
- Output file marine_taxa_table.csv is written and its schema matches the annotated table structure.

## Limitations

- No method, including this one, identifies all taxa in an ecosystem; the reproduced counts are lower bounds on true biodiversity.
- Amplification is subject to substantial primer biases and is almost certainly affected by inhibitors and variable abundances of template DNA, affecting which taxa are detected.
- Species-level assignment carries lower confidence than Family-or-higher assignment, which is why the table is agglomerated at Family or higher rank.
- eDNA sequences may come from dead or decaying biological matter and cannot resolve life history stages, so the marine taxa table reflects detectable DNA, not necessarily living community membership.

## Evidence

- [abstract] 663 taxonomic groups identified: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [readme] repository purpose: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
- [discussion] no method detects all taxa: "we note that no method—including this one—identifies all taxa in an ecosystem."
- [discussion] species-level confidence caveat: "Biodiversity fluctuations with time would ideally be observed on a species level, albeit the lower the taxonomy assignment, the lower the confidence in that assignment becomes."
