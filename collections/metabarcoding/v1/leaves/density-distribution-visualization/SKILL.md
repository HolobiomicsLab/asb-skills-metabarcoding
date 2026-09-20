---
name: density-distribution-visualization
description: Use when you need to visualize the shape of a per-sample distribution
  of a continuous community-level metric (e.g., copy-number-normalized ACN) across
  categorical environment or habitat classes, and check for multimodality.
when_to_use_negative:
- Input is a raw closed-reference OTU BIOM table that has not been divided by predicted
  copy numbers — the ACN density would be meaningless until normalize_by_copy_number.py
  is run
- Samples lack environment ontology labels or belong to only one environment class
  — no multi-class bimodality comparison (free-living/plant vs animal-associated)
  is possible
- Data are Deblur exact-sequence variants rather than Greengenes closed-reference
  OTUs — per-OTU predicted copy numbers from PICRUSt/Greengenes 13.8 do not apply
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: PICRUSt
  role: normalize_by_copy_number.py divides each Greengenes reference OTU's counts
    by its predicted 16S rRNA copy number to produce the normalized table used for
    weighted ACN
- name: QIIME1
  role: I/O for loading the closed-reference BIOM table and mapping file
- name: Python
  role: computes abundance-weighted per-sample ACN and renders density plots styled
    after Fig. 2d
- name: redbiom
  role: retrieves the closed-reference 16S V4 BIOM sample data and sample metadata
    for the EMP samples to visualize
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:73e163b87718094cb44bcd8c93a57d94002365926c94491235a5b981e38f7b26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - PICRUSt
  - Python
  - QIIME1
  - redbiom
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# density-distribution-visualization

## Summary

Produces abundance-weighted density plots of per-sample 16S rRNA gene copy number (ACN) across environment classes (free-living, plant-associated, animal-associated) to expose bimodal structure like EMP Fig. 2d, where free-living/plant samples centre on ACN ~2.2 and animal-associated samples on ~3.4.

## When to use

Use when you need to visualize the shape of a per-sample distribution of a continuous community-level metric (e.g., copy-number-normalized ACN) across categorical environment or habitat classes, and check for multimodality.

## When NOT to use

- Input is a raw closed-reference OTU BIOM table that has not been divided by predicted copy numbers — the ACN density would be meaningless until normalize_by_copy_number.py is run
- Samples lack environment ontology labels or belong to only one environment class — no multi-class bimodality comparison (free-living/plant vs animal-associated) is possible
- Data are Deblur exact-sequence variants rather than Greengenes closed-reference OTUs — per-OTU predicted copy numbers from PICRUSt/Greengenes 13.8 do not apply

## Inputs

- EMP 16S Release 1 closed-reference BIOM observation table (16S V4, picked closed-reference against Greengenes 13.8)
- Sample metadata mapping file with environment ontology labels
- PICRUSt 1.1.0 predicted 16S rRNA copy numbers for Greengenes reference OTUs

## Outputs

- acn_per_sample.tsv (one abundance-weighted mean 16S rRNA copy number per sample)
- Density plot of weighted ACN distributions per environment class, styled after Fig. 2d

## How to apply

Load the EMP 16S Release 1 closed-reference BIOM table (16S V4, picked closed-reference against Greengenes 13.8) and its mapping file with QIIME1/biom I/O and Python. Classify each sample with environment ontology labels into free-living, plant-associated, or animal-associated. Run PICRUSt 1.1.0 normalize_by_copy_number.py so each Greengenes reference OTU's counts are divided by its predicted 16S rRNA copy number, then compute each sample's relative-abundance-weighted mean copy number and write acn_per_sample.tsv. Plot the weighted ACN distributions as density curves per environment class with Python (Pandas/matplotlib styling after Fig. 2d), overlaying the three classes on one axis so the bimodal split between ~2.2 and ~3.4 peaks is visible.

## Related tools

- **PICRUSt** (normalize_by_copy_number.py divides each Greengenes reference OTU's counts by its predicted 16S rRNA copy number to produce the normalized table used for weighted ACN)
- **QIIME1** (I/O for loading the closed-reference BIOM table and mapping file)
- **Python** (computes abundance-weighted per-sample ACN and renders density plots styled after Fig. 2d)
- **redbiom** (retrieves the closed-reference 16S V4 BIOM sample data and sample metadata for the EMP samples to visualize) — https://github.com/biocore/redbiom

## Examples

```
export ctx=Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506; redbiom search metadata beer | head | redbiom fetch samples --context $ctx --output beer_example.biom
```

## Evaluation signals

- Free-living and plant-associated density curves peak near ACN 2.2, matching the reported Fig. 2d value
- Animal-associated density curve peaks near ACN 3.4, matching the reported Fig. 2d value
- The pooled weighted ACN distribution is visibly bimodal with two approximately equal-mass modes
- Each sample's relative-abundance weights sum to 1 before averaging, and the number of rows in acn_per_sample.tsv equals the number of retained BIOM samples
- Every plotted sample carries a valid environment ontology class assignment (free-living, plant-associated, or animal-associated)

## Limitations

- ACN values depend on PICRUSt-predicted copy numbers for Greengenes 13.8 closed-reference OTUs, so the metric is only meaningful for that closed-reference processing context
- The bimodal ~2.2 vs ~3.4 split was observed on EMP 16S Release 1 (~25,000 samples in 97 studies); other datasets or releases may not reproduce the same shape
- Sample-level weighting makes the density sensitive to upstream filtering choices (e.g., rarefaction to 5,000 observations per sample and exclusion of chloroplast-annotated sequences used in the paper's statistics)

## Evidence

- [results] the abundance-weighted ACN distribution was bimodal, with approximately half the dataset (free-living and plant-associated samples) centred on an ACN of 2.2: [evidence span withheld — anchor status: not_found]
- [methods] Run PICRUSt 1.1.0 normalize_by_copy_number.py on the closed-reference observation table to divide each Greengenes reference OTU's counts by its predicted 16S rRNA copy number: [evidence span withheld — anchor status: not_found]
- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
