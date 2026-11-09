---
name: 16s-copy-number-normalization
description: Use when you have a closed-reference Greengenes 13.8 OTU BIOM table (e.g.,
  EMP 16S V4) and need to correct 16S copy-number bias or compute per-sample average
  copy number (ACN) to compare host-associated vs. free-living communities.
when_to_use_negative:
- Input is a Deblur / exact-sequence table without Greengenes closed-reference OTU
  IDs — PICRUSt copy-number normalization requires Greengenes reference OTUs
- Data are already copy-number-normalized or you need shotgun functional profiles
  rather than 16S amplicon correction
- Samples lack environment ontology labels needed to assign free-living / plant-associated
  / animal-associated classes
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PICRUSt
  role: normalize_by_copy_number.py divides each Greengenes reference OTU's counts
    by its predicted 16S rRNA copy number (v1.1.0)
- name: QIIME1
  role: provides the closed-reference OTU BIOM table and biom I/O used as normalization
    input
- name: redbiom
  role: fetches closed-reference 16S V4 BIOM sample data and sample metadata from
    the Qiita EMP catalogue
  repo: https://github.com/biocore/redbiom
- name: Python
  role: computes per-sample abundance-weighted mean copy number and plots ACN density
    distributions per environment class
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:069ce86d46ed8bd42f7733c9b52f09eeae598d4474ec869f86b596248a205cb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - PICRUSt
  - Python
  - QIIME1
  - redbiom
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# 16S copy number normalization

## Summary

Normalizes a 16S amplicon OTU table by predicted 16S rRNA gene copy number using PICRUSt, then computes abundance-weighted average copy number (ACN) per sample to distinguish host-associated from free-living communities in EMP 16S Release 1 data.

## When to use

Use when you have a closed-reference Greengenes 13.8 OTU BIOM table (e.g., EMP 16S V4) and need to correct 16S copy-number bias or compute per-sample average copy number (ACN) to compare host-associated vs. free-living communities.

## When NOT to use

- Input is a Deblur / exact-sequence table without Greengenes closed-reference OTU IDs — PICRUSt copy-number normalization requires Greengenes reference OTUs
- Data are already copy-number-normalized or you need shotgun functional profiles rather than 16S amplicon correction
- Samples lack environment ontology labels needed to assign free-living / plant-associated / animal-associated classes

## Inputs

- EMP 16S Release 1 closed-reference BIOM observation table (16S V4, picked closed-reference against Greengenes 13.8)
- Sample metadata mapping file with environment ontology labels (free-living / plant-associated / animal-associated)

## Outputs

- Copy-number-normalized BIOM table (each Greengenes OTU's counts divided by its predicted 16S rRNA copy number)
- acn_per_sample.tsv: per-sample abundance-weighted mean 16S rRNA copy number
- Density plots of weighted ACN distributions per environment class (Fig. 2d style)

## How to apply

Load the EMP 16S Release 1 closed-reference BIOM observation table (16S V4, picked against Greengenes 13.8) with its sample metadata mapping file using QIIME1/biom I/O in Python. Assign samples with environment ontology labels to free-living, plant-associated, or animal-associated classes, then run PICRUSt 1.1.0 normalize_by_copy_number.py, which divides each Greengenes reference OTU's counts by its predicted 16S rRNA copy number to yield a normalized BIOM table. Compute each sample's abundance-weighted mean copy number (relative-abundance-weighted average over OTUs) and write acn_per_sample.tsv. Finally, plot weighted ACN density distributions per environment class and validate against the reported Fig. 2d pattern.

## Related tools

- **PICRUSt** (normalize_by_copy_number.py divides each Greengenes reference OTU's counts by its predicted 16S rRNA copy number (v1.1.0))
- **QIIME1** (provides the closed-reference OTU BIOM table and biom I/O used as normalization input)
- **redbiom** (fetches closed-reference 16S V4 BIOM sample data and sample metadata from the Qiita EMP catalogue) — https://github.com/biocore/redbiom
- **Python** (computes per-sample abundance-weighted mean copy number and plots ACN density distributions per environment class)

## Examples

```
export ctx=Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506; redbiom fetch samples --context $ctx --output emp_cr.biom
```

## Evaluation signals

- Abundance-weighted ACN distribution is bimodal: free-living and plant-associated samples center near ACN ≈ 2.2 and animal-associated samples near ACN ≈ 3.4 (Fig. 2d)
- Each normalized OTU count equals the original closed-reference count divided by that Greengenes OTU's predicted 16S rRNA copy number
- Every sample with environment ontology labels is assigned to exactly one of the three environment classes, and acn_per_sample.tsv contains one weighted mean per sample
- Retrieved samples match the expected context, e.g. the Greengenes-Illumina-16S-V4 closed-reference context rather than Deblur or Silva contexts

## Limitations

- Normalization is tied to the Greengenes 13.8 closed-reference OTU table; copy numbers are predicted per reference OTU, so de novo or Silva-based OTUs are not supported
- ACN is a relative-abundance-weighted mean, so it is sensitive to rarefaction depth and abundance skew (EMP supplementary analyses rarefied to 5,000 observations per sample)
- The bimodal 2.2 / 3.4 centers describe EMP 16S Release 1 (~25,000 samples, 97 studies); other datasets may show different ACN distributions

## Evidence

- [results] the abundance-weighted ACN distribution was bimodal, with approximately half the dataset (free-living and plant-associated samples) centred on an ACN of 2.2 and the other half (animal-associated samples) centred on an ACN of 3.4 (Fig. 2d): [evidence span withheld — anchor status: not_found]
- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [methods] picked OTUs using either Deblur or closed-reference against Greengenes 13.8 or Silva 123: [evidence span withheld — anchor status: not_found]
- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [readme] Get the closed reference OTU picking 16S V4 data for those samples (more on what `ctx` and `context` is in the longer examples below):: [evidence span withheld — anchor status: external_doc]
- [readme] Redbiom is a cache service for sample metadata and sample data.: [evidence span withheld — anchor status: external_doc]
