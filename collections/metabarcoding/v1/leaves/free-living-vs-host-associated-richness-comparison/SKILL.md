---
name: free-living-vs-host-associated-richness-comparison
description: Use when comparing per-sample richness between EMPO environment categories
  (host-associated vs free-living) on a rarefied, quality-filtered 16S feature table
  such as the EMP 16S Release 1 Deblur BIOM table.
when_to_use_negative:
- The feature table has not been rarefied to a common depth — raw richness differences
  would mostly reflect sequencing depth
- The goal is beta-diversity or taxonomic composition comparison rather than per-sample
  observed-richness ranking by environment category
- Samples lack EMPO (host-associated vs free-living) metadata annotations, making
  the grouping impossible
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3697
tools:
- name: Python
  role: Analysis language used to load the BIOM table and metadata and compute per-sample
    richness grouped by EMPO level
- name: Pandas
  role: DataFrame manipulation for joining per-sample richness with EMPO annotations
    and writing empo_richness_summary.csv
- name: Deblur
  role: Produces the exact-sequence 90-bp V4 tag sequences that define the richness
    features being counted
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:75f658555965f1f48213ac7c4a0bdba01ba11edb4772057325f799ec50dfb04a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Pandas
  - Python
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# free-living-vs-host-associated-richness-comparison

## Summary

Reproduces the EMP Fig. 2a alpha-diversity comparison by computing observed tag-sequence richness on the rarefied Deblur BIOM table and aggregating it by EMPO level to test whether host-associated communities are less rich than free-living ones.

## When to use

Use when comparing per-sample richness between EMPO environment categories (host-associated vs free-living) on a rarefied, quality-filtered 16S feature table such as the EMP 16S Release 1 Deblur BIOM table.

## When NOT to use

- The feature table has not been rarefied to a common depth — raw richness differences would mostly reflect sequencing depth
- The goal is beta-diversity or taxonomic composition comparison rather than per-sample observed-richness ranking by environment category
- Samples lack EMPO (host-associated vs free-living) metadata annotations, making the grouping impossible

## Inputs

- Rarefied, quality-filtered Deblur BIOM feature table (90-bp V4 tag sequences, 5,000 observations per sample)
- Merged EMP mapping/metadata file with EMPO environment annotations
- Filter specifications: mc2 minimum OTU count, chloroplast sequence exclusion

## Outputs

- Per-sample observed tag-sequence richness values
- empo_richness_summary.csv with one row per EMPO category and richness statistic (median/mean)

## How to apply

Load the rarefied, quality-filtered Deblur BIOM table (~23,828 analyzed samples from ~25,000 in EMP 16S Release 1) and the merged mapping file carrying EMPO annotations with Python/pandas and the biom library. Verify the table reflects release filters — mc2 (minimum sequence count in an OTU equals 2), chloroplast-annotated sequences excluded before statistics — and that each sample is rarefied to 5,000 observations so richness is depth-normalized. Compute observed tag-sequence richness as the count of distinct exact-sequence Deblur features present per sample, then group by EMPO level separating free-living and host-associated categories and compute median/mean richness per group. The rationale is that rarefaction removes sequencing-depth confounds so the free-living > host-associated ordering (except plant rhizosphere, which resembles free-living soil) reflects biology rather than effort.

## Related tools

- **Python** (Analysis language used to load the BIOM table and metadata and compute per-sample richness grouped by EMPO level)
- **Pandas** (DataFrame manipulation for joining per-sample richness with EMPO annotations and writing empo_richness_summary.csv)
- **Deblur** (Produces the exact-sequence 90-bp V4 tag sequences that define the richness features being counted) — https://github.com/cuttlefishh/deblur

## Examples

```
from biom import load_table; import pandas as pd; t = load_table('emp_deblur.biom'); obs = pd.Series({s: (t.data(s) > 0).sum() for s in t.ids('sample')}, name='observed_richness')
```

## Evaluation signals

- Summary CSV contains one row per EMPO environment category, each with a richness statistic
- Free-living EMPO groups rank above host-associated groups in median observed richness, matching the published Fig. 2a ordering
- Plant rhizosphere is the expected exception: its richness resembles free-living soil communities despite being host-associated
- Every analyzed sample's observation count equals 5,000, confirming depth normalization before richness computation
- Sample count in the analysis is ~23,828 of the ~25,000 in EMP 16S Release 1, consistent with the release filters

## Limitations

- Richness is restricted to samples rarefied to 5,000 observations, so samples below that depth are dropped from the ~25,000-sample release
- Chloroplast-annotated sequences and OTUs below mc2 are excluded before statistics, so counts differ from the raw Deblur table
- The comparison relies on EMPO annotation quality in the merged mapping file; mislabeled environment categories shift group medians

## Evidence

- [supplementary] rarefaction depth: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] chloroplast exclusion: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 filter: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [methods] release contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [readme] feature definition: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
