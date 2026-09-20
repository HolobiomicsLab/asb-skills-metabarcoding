---
name: rarefaction-depth-normalization
description: Use when computing alpha-diversity (e.g., observed tag-sequence richness)
  across EMP samples with heterogeneous sequencing depth, and depth-normalized comparisons
  are required before grouping by environment category.
when_to_use_negative:
- You are computing beta-diversity or compositional analyses where rarefaction discards
  data and alternatives are preferred.
- Samples have fewer than 5,000 observations and cannot be rarefied to the target
  depth without exclusion bias.
- The BIOM table still contains chloroplast-annotated sequences or has not had the
  mc2 minimum-count filter applied — apply those release filters first.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used with pandas/biom to load the rarefied BIOM table and metadata, compute
    per-sample richness, and aggregate by EMPO category
- name: Pandas
  role: Tabular handling of the merged mapping file and grouped summary statistics
    of richness per environment category
- name: Deblur
  role: Produces the exact-sequence (sub-OTU) feature table whose per-sample distinct-feature
    counts are the richness metric being depth-normalized
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:0903049185fcd185e431af81d543110a8a876bfaaca59639b3acf531872dbf97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rarefaction-depth-normalization

## Summary

Rarefies an EMP QC-filtered Deblur BIOM feature table to a fixed sequencing depth (5,000 observations per sample) so that per-sample alpha-diversity metrics such as observed tag-sequence richness are comparable across host-associated and free-living communities.

## When to use

Use when computing alpha-diversity (e.g., observed tag-sequence richness) across EMP samples with heterogeneous sequencing depth, and depth-normalized comparisons are required before grouping by environment category.

## When NOT to use

- You are computing beta-diversity or compositional analyses where rarefaction discards data and alternatives are preferred.
- Samples have fewer than 5,000 observations and cannot be rarefied to the target depth without exclusion bias.
- The BIOM table still contains chloroplast-annotated sequences or has not had the mc2 minimum-count filter applied — apply those release filters first.

## Inputs

- Quality-filtered Deblur BIOM feature table (EMP 16S Release 1, 90-bp V4 tag sequences)
- Merged metadata mapping file with EMPO environment annotations
- Rarefaction depth parameter (5,000 observations per sample)

## Outputs

- Depth-normalized (rarefied) BIOM table with 5,000 observations per sample
- Per-sample observed tag-sequence richness values
- Grouped richness summary per EMPO environment category (e.g., empo_richness_summary.csv)

## How to apply

Start from the rarefied, quality-filtered Deblur BIOM table and the merged metadata mapping file carrying EMPO annotations (~23,828 analyzed samples from ~25,000 in EMP 16S Release 1). Confirm the table reflects the release filters: mc2 (minimum sequence count in an OTU equals 2) and exclusion of chloroplast-annotated sequences before statistics. Confirm each sample is rarefied to exactly 5,000 observations per sample, since equal depth is the rationale for comparing richness across samples. Compute observed tag-sequence richness as the count of distinct exact-sequence (Deblur) features present per sample, then aggregate by EMPO free-living vs host-associated categories and compute summary statistics (median/mean). Write the grouped summary (e.g., empo_richness_summary.csv) and check the ordering against the published Fig. 2a pattern.

## Related tools

- **Python** (Used with pandas/biom to load the rarefied BIOM table and metadata, compute per-sample richness, and aggregate by EMPO category)
- **Pandas** (Tabular handling of the merged mapping file and grouped summary statistics of richness per environment category)
- **Deblur** (Produces the exact-sequence (sub-OTU) feature table whose per-sample distinct-feature counts are the richness metric being depth-normalized) — https://github.com/cuttlefishh/deblur

## Examples

```
import biom, pandas as pd; table = biom.load_table('emp_deblur_mc2_no_chloroplast.biom'); rich = pd.Series({s: (table.data(s) > 0).sum() for s in table.ids()}); summary = rich.groupby(metadata['empo_3']).describe(); summary.to_csv('empo_richness_summary.csv')
```

## Evaluation signals

- Every retained sample has exactly 5,000 observations in the rarefied table (column sums equal 5,000).
- The analyzed sample count matches the release scale (~23,828 of ~25,000 samples).
- Richness is defined as the count of distinct exact-sequence features present per sample, matching the Deblur sub-OTU definition.
- Grouped summary contains one row per EMPO environment category with a richness statistic.
- Free-living groups (except plant rhizosphere) rank above host-associated groups, reproducing the published Fig. 2a ordering.

## Limitations

- Rarefaction to 5,000 observations discards reads from deeper-sequenced samples and excludes shallower samples, reducing usable data.
- Richness comparisons are only valid because chloroplast-annotated sequences were excluded and the mc2 filter applied before statistics.
- The plant rhizosphere exception (host-associated but free-living-like richness) means category-level generalizations will not hold for every EMPO class.

## Evidence

- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Chloroplast exclusion before statistics: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] Deblur tag sequences: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] EMP 16S Release 1 contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
