---
name: alpha-diversity-richness-estimation
description: Use when comparing microbial richness across environment categories or
  environmental gradients in the EMP 16S Release 1 Deblur BIOM table, and you need
  depth-normalized observed tag-sequence counts (Fig. 2a/2b reproduction).
when_to_use_negative:
- The feature table is unrarefied or unfiltered (no mc2, chloroplast sequences retained)
  — richness comparisons would not be depth-normalized and would not match the release
  statistics
- The analysis goal is beta-diversity/compositional turnover rather than within-sample
  richness counting
- Data are clustered OTUs rather than Deblur exact sequences — the paper's cross-study
  comparability relies on exact-sequence features, not clustered OTUs
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Python
  role: Load BIOM table and metadata, compute per-sample observed richness, group
    and summarize statistics
- name: Pandas
  role: DataFrame handling of the mapping file, binning pH/temperature, and writing
    grouped summaries to CSV
- name: Deblur
  role: Produces the exact-sequence feature table (90-bp V4 tag sequences) on which
    richness is counted
  repo: https://github.com/cuttlefishh/deblur
- name: QIIME1
  role: Merge BIOM tables and mapping files to assemble the analysis-ready table and
    metadata
- name: QIIME2
  role: Merge BIOM tables and mapping files as an alternative to QIIME1
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:3dca7110da7435c5633af5aa16b97585252966c19a40b7ede1338f8a8f1dffd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Pandas
  - Python
  - QIIME1
  - QIIME2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# alpha-diversity-richness-estimation

## Summary

Compute per-sample alpha-diversity richness (count of distinct exact tag sequences present) on the EMP 16S Release 1 Deblur feature table after applying release filters (mc2, chloroplast removal, rarefaction to 5,000 observations/sample), then aggregate richness by EMPO environment category or by environmental gradients such as pH and temperature.

## When to use

Use when comparing microbial richness across environment categories or environmental gradients in the EMP 16S Release 1 Deblur BIOM table, and you need depth-normalized observed tag-sequence counts (Fig. 2a/2b reproduction).

## When NOT to use

- The feature table is unrarefied or unfiltered (no mc2, chloroplast sequences retained) — richness comparisons would not be depth-normalized and would not match the release statistics
- The analysis goal is beta-diversity/compositional turnover rather than within-sample richness counting
- Data are clustered OTUs rather than Deblur exact sequences — the paper's cross-study comparability relies on exact-sequence features, not clustered OTUs

## Inputs

- EMP 16S Release 1 QC-filtered Deblur BIOM feature table (90-bp V4 exact tag sequences)
- Merged metadata mapping file with EMPO annotations, pH, and temperature fields
- Filter specifications: mc2, chloroplast exclusion, rarefaction depth 5,000

## Outputs

- Per-sample observed richness values keyed to sample IDs
- empo_richness_summary.csv (one row per EMPO environment category with a richness statistic)
- richness_gradient_laplace_fits.png (binned 99th-percentile richness vs pH/temperature with Laplace and Gaussian fits)

## How to apply

Load the QC-filtered Deblur BIOM feature table and merged metadata mapping file (~23,828 analyzed samples of ~25,000 in the release) with Python/pandas and the biom library. Confirm the table reflects release filters: mc2 (minimum sequence count in an OTU equal to 2), exclusion of chloroplast-annotated sequences, and rarefaction of each sample to 5,000 observations so richness comparisons are depth-normalized. Compute observed richness as the count of distinct 90-bp V4 exact-sequence features present in each sample. For environment-type comparisons, group per-sample richness by EMPO level (free-living vs host-associated) and compute median/mean per group; for gradient analyses, bin samples by pH into 100 equal bins and temperature into 120 equal bins, compute the 99th-percentile richness per bin, and fit richness = a*exp(-|x-x0|/b) (Laplace) by nonlinear least squares with scipy.optimize.curve_fit, comparing against a Gaussian fit.

## Related tools

- **Python** (Load BIOM table and metadata, compute per-sample observed richness, group and summarize statistics)
- **Pandas** (DataFrame handling of the mapping file, binning pH/temperature, and writing grouped summaries to CSV)
- **Deblur** (Produces the exact-sequence feature table (90-bp V4 tag sequences) on which richness is counted) — https://github.com/cuttlefishh/deblur
- **QIIME1** (Merge BIOM tables and mapping files to assemble the analysis-ready table and metadata)
- **QIIME2** (Merge BIOM tables and mapping files as an alternative to QIIME1)

## Examples

```
import biom; t = biom.load_table('deblur_mc2_rare5000.biom'); richness = {s: int((t.data(s) > 0).sum()) for s in t.ids()}
```

## Evaluation signals

- Summary table contains exactly one row per EMPO environment category with a richness statistic
- Free-living EMPO groups (except plant rhizosphere) rank above host-associated groups in richness, matching published Fig. 2a ordering
- Laplace fit peak locations fall near neutral pH (~7) and ~10 °C, consistent with Fig. 2b
- Laplace fit outperforms the Gaussian fit on relative goodness of fit (R²/residual sum of squares) over the same binned data
- Sample count after filtering is ~23,828 samples from the ~25,000 in EMP 16S Release 1, with each sample at 5,000 rarefied observations

## Limitations

- Comparability depends on coordinated protocols and exact sequences (Deblur) rather than clustered OTUs; applying the skill to heterogeneous or clustered tables undermines cross-study inference
- Richness comparisons are valid only at the fixed 5,000-observation rarefaction depth; shallower or uneven depths bias observed counts
- The rarefied table covers ~23,828 of ~25,000 samples; only ~27,751 samples have metadata for any given field, so gradient analyses (pH, temperature) use subsets

## Evidence

- [supplementary] Rarefaction depth: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Chloroplast exclusion: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] Feature type: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] Table assembly: [evidence span withheld — anchor status: not_found]
