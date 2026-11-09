---
name: rarefaction-normalization-of-16s-counts
description: Use when comparing environment distributions or alpha diversity across
  EMP samples processed in different studies, where BIOM tables have heterogeneous
  depths and counts must first be equalized to 5,000 observations per sample.
when_to_use_negative:
- The input table is already rarefied or contains only samples with <=5,000 observations
  of comparable depth, where additional rarefaction would discard data unnecessarily
- The analysis requires absolute abundances or differential-abundance methods designed
  for compositional counts rather than equalized depth
- Working with shotgun metagenomic or ITS/metabolomics data from EMP500, where the
  5,000-observation 16S rarefaction depth does not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics: []
tools:
- name: QIIME1
  role: Merges the BIOM tables and mapping files before rarefaction and filtering
  repo: https://github.com/biocore/emp/tree/master/methods
- name: QIIME2
  role: Alternative tool for merging the BIOM tables and mapping files prior to normalization
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Deblur
  role: Produces the exact-sequence (90-bp V4) tag-sequence BIOM tables that are rarefied
    and analyzed
  repo: https://github.com/cuttlefishh/deblur
- name: Python
  role: Used with Pandas to implement rarefaction, filtering, and downstream entropy
    computation
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Pandas
  role: DataFrame manipulation of the rarefied count table and mapping metadata
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:9d54e4a3b9afb51591ef94929b32aadb4d6b3ebc8a2cde50021e67a3327097ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Pandas
  - Python
  - QIIME1
  - QIIME2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# rarefaction-normalization-of-16s-counts

## Summary

Normalizes EMP 16S amplicon count tables (Deblur tag-sequence or closed-reference OTU BIOM tables) by rarefying to a fixed 5,000 observations per sample, so cross-study Shannon-entropy and diversity analyses are not biased by unequal sequencing depth.

## When to use

Use when comparing environment distributions or alpha diversity across EMP samples processed in different studies, where BIOM tables have heterogeneous depths and counts must first be equalized to 5,000 observations per sample.

## When NOT to use

- The input table is already rarefied or contains only samples with <=5,000 observations of comparable depth, where additional rarefaction would discard data unnecessarily
- The analysis requires absolute abundances or differential-abundance methods designed for compositional counts rather than equalized depth
- Working with shotgun metagenomic or ITS/metabolomics data from EMP500, where the 5,000-observation 16S rarefaction depth does not apply

## Inputs

- EMP 16S Release 1 Deblur tag-sequence BIOM table (90-bp V4 sequences from ftp://ftp.microbio.me/emp/release1)
- EMP 16S Release 1 closed-reference OTU BIOM tables (Greengenes 13.8 / Silva 123)
- Merged quality-filtered mapping files with environment-category metadata

## Outputs

- Rarefied BIOM table with exactly 5,000 observations per sample
- Chloroplast-excluded, mc2-filtered count table ready for entropy/diversity computation

## How to apply

After merging the EMP 16S Release 1 BIOM tables and quality-filtered mapping files with QIIME1 or QIIME2, rarefy each sample to exactly 5,000 observations per sample before computing any statistics. While rarefied, apply the study's standard filters: exclude sequences annotated as chloroplast, and apply mc2 filtering (minimum sequence count in an OTU equals 2). The rarefied table is then the input for downstream per-sequence analyses, e.g. computing Shannon entropy H = -sum(p_i * log2 p_i) over environment categories for each exact Deblur V4 tag sequence, or for counts collapsed to genera or phylogenetic subtrees. Rationale: samples in EMP 16S Release 1 span 97 studies with very different sequencing depths, so without rarefaction, deeply sequenced samples would dominate entropy and diversity estimates.

## Related tools

- **QIIME1** (Merges the BIOM tables and mapping files before rarefaction and filtering) — https://github.com/biocore/emp/tree/master/methods
- **QIIME2** (Alternative tool for merging the BIOM tables and mapping files prior to normalization) — https://github.com/biocore/emp/tree/master/methods
- **Deblur** (Produces the exact-sequence (90-bp V4) tag-sequence BIOM tables that are rarefied and analyzed) — https://github.com/cuttlefishh/deblur
- **Python** (Used with Pandas to implement rarefaction, filtering, and downstream entropy computation) — https://github.com/biocore/emp/tree/master/methods
- **Pandas** (DataFrame manipulation of the rarefied count table and mapping metadata)

## Examples

```
python -c "import biom, numpy as np; t = biom.load_table('emp_deblur.biom'); t.filter(lambda v,i,m: v.sum()==0 or True); t.norm()"
```

## Evaluation signals

- Every retained sample has exactly 5,000 total observations in the rarefied table (column sums all equal 5,000)
- No chloroplast-annotated sequences remain in the filtered table before statistics are computed
- Every OTU/feature passing mc2 filtering has a minimum sequence count of 2 in at least one sample
- All downstream entropy values fall between 0 and log2(number of environment categories)
- Rarefied per-sample totals match across studies despite the 97 original studies having heterogeneous depths

## Limitations

- Rarefying to 5,000 observations discards samples with fewer than 5,000 reads and truncates deeper samples, losing information
- The 5,000-observation depth is specific to EMP 16S Release 1 (27,751 samples with metadata; ~25,000 samples in 97 studies) and may not suit shallower or deeper datasets
- mc2 filtering combined with rarefaction further reduces feature count, which can affect low-abundance tag sequences

## Evidence

- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [methods] mc2 filtering definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] Merge BIOM tables with QIIME: [evidence span withheld — anchor status: not_found]
- [methods] EMP 16S Release 1 scope: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [readme] Deblur tag sequences from README context: "EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published"
