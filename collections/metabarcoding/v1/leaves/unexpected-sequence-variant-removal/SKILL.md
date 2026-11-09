---
name: unexpected-sequence-variant-removal
description: Use when a mock-community ASV or OTU feature table (e.g. from DADA2,
  Deblur, or VSEARCH clustering of 181 bp COI amplicons) still retains unexpected
  'miss' or partial mock variants after denoising, and abundance-based filtering must
  remove them.
when_to_use_negative:
- Input is raw demultiplexed reads or FASTQ files with no feature table yet — denoising/clustering
  must precede this filtering step
- The dataset has no biological mock community, so the per-library fixed integer for
  the extra filter cannot be empirically derived from 'miss' variant maxima
- The goal is filtering a reference database (e.g. by clustering radius or taxonomic
  completeness), not abundance filtering of a feature table
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: qiime2R
  role: R package used to load the denoised/clustering mock-community ASV frequency
    (.qza) table prior to filtering
- name: Tidyverse
  role: R toolkit for the per-sample/per-library filtering operations (dropping single-sample
    ASVs, integer subtraction, zeroing counts)
- name: pandas
  role: Alternative Python loader/manipulator for the feature table if working outside
    R
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:63be16e0c3d9d214a0608c31c0e2afa2b46eeb052fad04a4ab3105ffa1b0c623
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2R
  - R
  - Tidyverse
  - pandas
  - qiime2R
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# unexpected-sequence-variant-removal

## Summary

A two-stage abundance filtering procedure from O'Rourke et al. (2020, Ecology and Evolution) that removes unexpected mock-community sequence variants (e.g. 'miss' and partial ASVs) from denoised/clustering COI metabarcoding feature tables by combining a standard low-abundance filter with a per-library fixed-integer subtraction.

## When to use

Use when a mock-community ASV or OTU feature table (e.g. from DADA2, Deblur, or VSEARCH clustering of 181 bp COI amplicons) still retains unexpected 'miss' or partial mock variants after denoising, and abundance-based filtering must remove them.

## When NOT to use

- Input is raw demultiplexed reads or FASTQ files with no feature table yet — denoising/clustering must precede this filtering step
- The dataset has no biological mock community, so the per-library fixed integer for the extra filter cannot be empirically derived from 'miss' variant maxima
- The goal is filtering a reference database (e.g. by clustering radius or taxonomic completeness), not abundance filtering of a feature table

## Inputs

- Denoised/clustering mock-community ASV frequency table (QIIME2 .qza artifact, BIOM, or TSV)
- List or taxonomy strings identifying unexpected ('miss', partial) mock sequence variants
- Sample-to-library mapping to define per-library mock samples

## Outputs

- Filtered feature table (BIOM/TSV) in which every 'miss'-categorized mock ASV has zero abundance
- Set of retained samples each meeting the ≥5,000 filtered-read minimum

## How to apply

Load the denoised/clustering mock-community ASV frequency table (QIIME2 .qza via QIIME2R, or BIOM/TSV via pandas) from the prior task. Apply the standard filter: (a) drop any sequence variant observed in just one sample across the entire dataset, and (b) retain only samples with ≥5,000 total filtered reads. Then, on a per-library basis, determine the maximum count value observed for 'miss' (unexpected) mock sequence variants in each library-specific mock sample. Apply the extra filter: subtract that fixed integer from each element of the feature table per library, zeroing any resulting negative or zero counts. Verify that all unexpected 'miss' mock ASVs are removed, then export the filtered feature table in BIOM/TSV format as the canonical output.

## Related tools

- **qiime2R** (R package used to load the denoised/clustering mock-community ASV frequency (.qza) table prior to filtering)
- **Tidyverse** (R toolkit for the per-sample/per-library filtering operations (dropping single-sample ASVs, integer subtraction, zeroing counts))
- **pandas** (Alternative Python loader/manipulator for the feature table if working outside R)

## Examples

```
ft <- qiime2R::read_qza("mock_asv_table.qza")$data; ft <- ft[rowSums(ft > 0) >= 2, ]; ft <- ft[, colSums(ft) >= 5000]
```

## Evaluation signals

- Every 'miss'-categorized mock ASV has zero abundance in the exported filtered table
- All retained samples meet the ≥5,000 total filtered reads minimum
- No retained sequence variant is observed in just one sample across the entire dataset
- All values in the filtered feature table are non-negative integers after the per-library subtraction
- Diffing pre- and post-filter tables shows reduced numbers of unexpected mock ASVs (partial and miss variants) while expected mock ASVs are retained

## Limitations

- The extra filter's fixed integer is derived from per-library maxima of 'miss' mock variants, so it depends on having mock-community controls representative of the libraries being filtered
- The choice of both denoising program and filtering can influence the number of mock ASVs observed in the dataset, so results differ between DADA2/Deblur and clustering outputs
- Low-abundance removal largely mitigates, but does not eliminate, the error-rate differences between denoising and traditional clustering methods

## Evidence

- [methods] a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [intro] Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018): "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
- [abstract] although these differences are largely mitigated by removing low‐abundance sequence variants: "although these differences are largely mitigated by removing low‐abundance sequence variants"
