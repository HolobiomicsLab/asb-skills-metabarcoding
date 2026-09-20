---
name: abundance-weighted-community-metric-computation
description: Use when a feature table (BIOM) and per-feature values (e.g., predicted
  16S rRNA copy number) must be combined into one abundance-weighted per-sample score
  for comparing environment classes such as animal-associated vs. free-living.
when_to_use_negative:
- The feature table is not closed-reference picked against Greengenes 13.8, so PICRUSt
  copy-number predictions cannot be joined to OTU IDs
- The analysis goal is sequence-variant level (e.g., Deblur exact sequences) rather
  than Greengenes reference OTU-level functional inference
- Samples lack environment ontology labels, so the free-living / plant-associated
  / animal-associated classes cannot be assigned
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PICRUSt
  role: normalize_by_copy_number.py divides each Greengenes reference OTU's counts
    by its predicted 16S rRNA copy number to yield the normalized BIOM table
- name: QIIME1
  role: biom/mapping-file I/O for the EMP closed-reference observation table used
    in the weighted metric computation
- name: Python
  role: computes per-sample abundance-weighted mean copy number and plots per-class
    density distributions (with Pandas)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:ebcaf6e345a2fc7546da0b01d51f6f520f85efc12099707de8d54d168f232836
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PICRUSt
  - Python
  - QIIME1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# abundance-weighted-community-metric-computation

## Summary

Computes per-sample community-level metrics as abundance-weighted averages over OTU features, exemplified by the average 16S rRNA copy number (ACN) on the EMP 16S Release 1 closed-reference Greengenes 13.8 table normalized with PICRUSt, to distinguish animal-associated from free-living/plant-associated communities.

## When to use

Use when a feature table (BIOM) and per-feature values (e.g., predicted 16S rRNA copy number) must be combined into one abundance-weighted per-sample score for comparing environment classes such as animal-associated vs. free-living.

## When NOT to use

- The feature table is not closed-reference picked against Greengenes 13.8, so PICRUSt copy-number predictions cannot be joined to OTU IDs
- The analysis goal is sequence-variant level (e.g., Deblur exact sequences) rather than Greengenes reference OTU-level functional inference
- Samples lack environment ontology labels, so the free-living / plant-associated / animal-associated classes cannot be assigned

## Inputs

- EMP 16S Release 1 closed-reference BIOM observation table (16S V4, Greengenes 13.8)
- Sample metadata mapping file with environment ontology labels
- PICRUSt Greengenes 13.8 predicted 16S rRNA copy-number table

## Outputs

- Normalized BIOM table (OTU counts divided by predicted 16S rRNA copy number)
- acn_per_sample.tsv with one abundance-weighted ACN value per sample
- Density plot of weighted ACN distributions per environment class (Fig. 2d style)

## How to apply

Load the EMP 16S Release 1 closed-reference BIOM observation table (16S V4, picked closed-reference against Greengenes 13.8) with its mapping file using QIIME1/biom I/O in Python, then assign each sample with an environment ontology label to free-living, plant-associated, or animal-associated. Run PICRUSt 1.1.0 normalize_by_copy_number.py to divide each Greengenes reference OTU's counts by its predicted 16S rRNA copy number, giving a normalized BIOM table. For each sample, compute the relative-abundance-weighted mean copy number over OTUs and write acn_per_sample.tsv. Validate by plotting per-class density distributions and confirming the free-living and plant-associated peaks near 2.2 and the animal-associated peak near 3.4, matching the paper's Fig. 2d.

## Related tools

- **PICRUSt** (normalize_by_copy_number.py divides each Greengenes reference OTU's counts by its predicted 16S rRNA copy number to yield the normalized BIOM table)
- **QIIME1** (biom/mapping-file I/O for the EMP closed-reference observation table used in the weighted metric computation)
- **Python** (computes per-sample abundance-weighted mean copy number and plots per-class density distributions (with Pandas))

## Examples

```
normalize_by_copy_number.py -i emp_cr_otu_table.biom -o acn_normalized_table.biom; python compute_acn.py --table acn_normalized_table.biom --map emp_map.txt --output acn_per_sample.tsv
```

## Evaluation signals

- The free-living and plant-associated ACN distribution centers fall near 2.2
- The animal-associated ACN distribution center falls near 3.4
- The abundance-weighted ACN distribution is bimodal, with roughly half the dataset in each mode
- acn_per_sample.tsv contains one weighted mean per sample, computed as a relative-abundance-weighted average over all OTUs
- The normalized table equals the original BIOM table with each OTU row divided by its predicted copy number

## Limitations

- The metric requires closed-reference OTU picking against Greengenes 13.8 (as in EMP 16S Release 1); other reference databases such as Silva 123 or Deblur exact sequences are not covered
- Copy numbers are PICRUSt predictions, so ACN values are estimates rather than direct measurements
- Interpretation depends on correct assignment of environment ontology labels to the three environment classes

## Evidence

- [methods] Greengenes 13.8 closed-reference OTU picking: [evidence span withheld — anchor status: not_found]
- [supplementary] Fig. 2d ACN peaks: [evidence span withheld — anchor status: not_found]
- [readme] redbiom retrieval of closed-reference 16S V4 data: [evidence span withheld — anchor status: external_doc]
