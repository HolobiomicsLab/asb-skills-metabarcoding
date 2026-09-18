---
name: picrust-functional-prediction-from-amplicon-data
description: Use when you have a Greengenes 13.8 closed-reference 16S BIOM table with
  environment ontology metadata and need to test functional/gene-copy-number differences
  between sample classes (e.g., animal-associated vs free-living).
when_to_use_negative:
- The feature table comes from Deblur exact-sequence (de novo) OTU picking rather
  than closed-reference picking against Greengenes 13.8 — PICRUSt copy-number normalization
  requires reference OTUs with predicted copy numbers
- Data are already functional profiles (e.g., shotgun metagenomics such as EMP500)
  where rRNA copy-number prediction is unnecessary
- Samples lack environment ontology labels, so the animal-associated / free-living
  / plant-associated class assignment cannot be made
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_0622
tools:
- name: PICRUSt
  role: normalize_by_copy_number.py divides each Greengenes reference OTU's counts
    by its predicted 16S rRNA copy number to yield the normalized BIOM table
- name: QIIME1
  role: Loads/merges the closed-reference BIOM table and mapping file used as PICRUSt
    input
- name: redbiom
  role: Retrieves the closed-reference 16S V4 BIOM sample data (context Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506)
    from the Qiita-backed EMP catalogue
  repo: https://github.com/biocore/redbiom
- name: Python
  role: Computes per-sample abundance-weighted ACN and plots density distributions
    per environment class
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:d22ce22df8c4cb9a4c0f321262b7c601455950231f208797d645d48fe59c30a2
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
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# picrust-functional-prediction-from-amplicon-data

## Summary

Uses PICRUSt 1.1.0 normalize_by_copy_number.py to normalize EMP 16S V4 closed-reference (Greengenes 13.8) OTU counts by predicted 16S rRNA copy number, enabling abundance-weighted average copy number (ACN) comparisons across environment classes.

## When to use

Use when you have a Greengenes 13.8 closed-reference 16S BIOM table with environment ontology metadata and need to test functional/gene-copy-number differences between sample classes (e.g., animal-associated vs free-living).

## When NOT to use

- The feature table comes from Deblur exact-sequence (de novo) OTU picking rather than closed-reference picking against Greengenes 13.8 — PICRUSt copy-number normalization requires reference OTUs with predicted copy numbers
- Data are already functional profiles (e.g., shotgun metagenomics such as EMP500) where rRNA copy-number prediction is unnecessary
- Samples lack environment ontology labels, so the animal-associated / free-living / plant-associated class assignment cannot be made

## Inputs

- EMP 16S Release 1 closed-reference BIOM observation table (16S V4, picked closed-reference against Greengenes 13.8)
- Sample metadata mapping file with environment ontology labels (free-living / plant-associated / animal-associated)
- Greengenes 13.8 reference OTU table with predicted 16S rRNA copy numbers (used by PICRUSt)

## Outputs

- Copy-number-normalized BIOM table from normalize_by_copy_number.py
- acn_per_sample.tsv with each sample's relative-abundance-weighted mean 16S rRNA copy number
- Density plots of weighted ACN distributions per environment class (Fig. 2d style)

## How to apply

Load the EMP 16S Release 1 closed-reference BIOM observation table (16S V4, picked closed-reference against Greengenes 13.8) together with its sample metadata mapping file using QIIME1/biom I/O in Python. Assign each sample carrying environment ontology labels to one of three classes: free-living, plant-associated, or animal-associated. Run PICRUSt 1.1.0 normalize_by_copy_number.py to divide each Greengenes reference OTU's counts by its predicted 16S rRNA copy number, producing a normalized BIOM table; the rationale is that amplicon OTU abundances are biased by each taxon's rRNA operon copy number, so normalization recovers approximate organismal (and functional) proportions. For each sample compute the abundance-weighted mean copy number as the relative-abundance-weighted average over OTUs and write acn_per_sample.tsv. Validate by checking the per-class density distributions of weighted ACN: free-living and plant-associated centers near 2.2, animal-associated center near 3.4, with an overall bimodal shape.

## Related tools

- **PICRUSt** (normalize_by_copy_number.py divides each Greengenes reference OTU's counts by its predicted 16S rRNA copy number to yield the normalized BIOM table)
- **QIIME1** (Loads/merges the closed-reference BIOM table and mapping file used as PICRUSt input)
- **redbiom** (Retrieves the closed-reference 16S V4 BIOM sample data (context Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506) from the Qiita-backed EMP catalogue) — https://github.com/biocore/redbiom
- **Python** (Computes per-sample abundance-weighted ACN and plots density distributions per environment class)

## Examples

```
redbiom fetch samples --ctx Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506 --output emp.biom; normalize_by_copy_number.py -i emp.biom -o normalized.biom
```

## Evaluation signals

- Normalized BIOM table entries equal original closed-reference OTU counts divided by the Greengenes 13.8 predicted 16S rRNA copy number for each OTU
- acn_per_sample.tsv values are within the plausible rRNA operon range (~1–6) and equal the relative-abundance-weighted average over OTUs
- Weighted ACN density distribution is bimodal across the full EMP dataset
- Free-living and plant-associated ACN distribution centers fall near 2.2 and animal-associated centers near 3.4, matching reported Fig. 2d values

## Limitations

- Requires closed-reference OTU picking against Greengenes 13.8; the EMP meta-analysis also used Deblur exact-sequence picking, which this copy-number normalization route does not cover
- Copy numbers are PICRUSt 1.1.0 predictions, so ACN is an estimate tied to that reference and version
- Confounds must be handled before statistics: in the EMP analyses chloroplast-annotated sequences were excluded and samples rarefied to 5,000 observations per sample

## Evidence

- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples"
- [methods] picked OTUs using either Deblur or closed-reference against Greengenes 13.8 or Silva 123: [evidence span withheld — anchor status: not_found]
- [supplementary] Samples were rarefied to 5,000 observations per sample: "Samples were rarefied to 5,000 observations per sample."
- [readme] retrieval of sample data into BIOM: "Redbiom is a command-line tool that allows users to query the Qiita database, including EMP studies."
- [readme] Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506: [evidence span withheld — anchor status: external_doc]
