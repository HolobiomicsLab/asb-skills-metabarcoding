---
name: environment-ontology-stratification
description: Use when sample metadata carries environment ontology labels that must
  be mapped to coarse habitat classes before comparing community-level metrics (e.g.,
  weighted ACN, alpha/beta diversity) across animal-associated vs free-living or plant-associated
  EMP samples.
when_to_use_negative:
- Samples lack environment ontology labels in the mapping file, so they cannot be
  assigned to a habitat class
- Input is shotgun metagenomic or other non-16S-V4 data not processed against Greengenes
  13.8 closed-reference, where PICRUSt copy-number normalization is not applicable
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: QIIME1
  role: Used to read/merge the BIOM tables and mapping files that supply sample metadata
    for environment class assignment
  repo: https://github.com/biocore/emp
- name: Python
  role: Used with biom I/O to load the observation table and metadata, assign environment
    classes, compute per-sample weighted ACN, and plot distributions
- name: PICRUSt
  role: normalize_by_copy_number.py normalizes the closed-reference table by predicted
    16S rRNA copy number after stratification
- name: redbiom
  role: Can retrieve sample data and metadata for EMP contexts (e.g., Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4)
    to populate the analysis inputs
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:29815a4e301e251bcad0709314fed22748f21d0ba33d1896ac3ae355e0567fb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/emp
  tools:
  - PICRUSt
  - Python
  - QIIME1
  - redbiom
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# environment-ontology-stratification

## Summary

Stratifies EMP samples into environment classes (free-living, plant-associated, animal-associated) using environment ontology metadata labels, enabling environment-stratified comparisons such as abundance-weighted average 16S rRNA copy number (ACN) via PICRUSt on Greengenes 13.8 closed-reference tables.

## When to use

Use when sample metadata carries environment ontology labels that must be mapped to coarse habitat classes before comparing community-level metrics (e.g., weighted ACN, alpha/beta diversity) across animal-associated vs free-living or plant-associated EMP samples.

## When NOT to use

- Samples lack environment ontology labels in the mapping file, so they cannot be assigned to a habitat class
- Input is shotgun metagenomic or other non-16S-V4 data not processed against Greengenes 13.8 closed-reference, where PICRUSt copy-number normalization is not applicable

## Inputs

- EMP 16S Release 1 sample metadata mapping file with environment ontology labels
- EMP 16S Release 1 closed-reference BIOM observation table (16S V4, Greengenes 13.8)

## Outputs

- Sample-to-environment-class assignment table (free-living / plant-associated / animal-associated)
- acn_per_sample.tsv: per-sample abundance-weighted mean 16S rRNA copy number
- Density plots of weighted ACN distributions per environment class (Fig. 2d style)

## How to apply

Load the EMP 16S Release 1 mapping file alongside its closed-reference BIOM table (16S V4, picked closed-reference against Greengenes 13.8) using QIIME1/biom I/O in Python. Inspect the environment ontology metadata field on each sample and assign every sample carrying a label to exactly one of three classes: free-living, plant-associated, or animal-associated. Keep sample IDs consistent with the BIOM table so per-sample downstream computations (e.g., PICRUSt 1.1.0 normalize_by_copy_number.py followed by abundance-weighted mean copy number) can be grouped by class. The rationale is that environment class is the key stratification variable underlying the paper's finding of a bimodal weighted ACN distribution (free-living/plant-associated centred near 2.2, animal-associated near 3.4, Fig. 2d). Validate assignments by checking that class-wise ACN density plots reproduce those reported centers.

## Related tools

- **QIIME1** (Used to read/merge the BIOM tables and mapping files that supply sample metadata for environment class assignment) — https://github.com/biocore/emp
- **Python** (Used with biom I/O to load the observation table and metadata, assign environment classes, compute per-sample weighted ACN, and plot distributions)
- **PICRUSt** (normalize_by_copy_number.py normalizes the closed-reference table by predicted 16S rRNA copy number after stratification)
- **redbiom** (Can retrieve sample data and metadata for EMP contexts (e.g., Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4) to populate the analysis inputs) — https://github.com/biocore/redbiom

## Examples

```
normalize_by_copy_number.py -i otu_table.biom -o normalized_otus.biom
```

## Evaluation signals

- Every sample in the output assignment table carries exactly one environment class and matches a sample ID present in the BIOM table
- Free-living and plant-associated weighted ACN distributions centre near 2.2, and animal-associated near 3.4, reproducing Fig. 2d
- The weighted ACN distribution is bimodal, with roughly half the dataset in each mode
- acn_per_sample.tsv contains a finite weighted mean copy number for every stratified sample (no missing/unparsed metadata rows)

## Limitations

- Samples without environment ontology labels must be excluded from stratification, reducing sample coverage
- ACN values depend on PICRUSt predictions against Greengenes 13.8; accuracy is limited by reference database coverage
- Results are specific to 16S V4 closed-reference processing; re-running on other contexts (e.g., Deblur) is not directly comparable

## Evidence

- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [results] weighted ACN bimodality: [evidence span withheld — anchor status: not_found]
- [methods] closed-reference Greengenes context: [evidence span withheld — anchor status: not_found]
- [readme] redbiom closed-reference retrieval: [evidence span withheld — anchor status: external_doc]
