---
name: feature-table-low-abundance-filtering
description: Use when a denoised or clustered COI metabarcoding feature table (e.g.,
  from DADA2, Deblur, or VSEARCH) still contains singleton or mock-derived spurious
  ASVs that must be removed before diversity analyses.
when_to_use_negative:
- Input is already a fully filtered feature table that has had singletons removed
  and samples rarefied or read-depth thresholded — reapplying would double-filter
- Data are not amplicon ASV/OTU count tables (e.g., shotgun metagenomic gene counts
  or taxonomy-level tables), since the mock-guided 'miss' subtraction is specific
  to sequence variants
- No mock-community control exists in the run, making the per-library 'miss' max-count
  integer impossible to derive
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2R
  role: Import the QIIME 2-derived feature table into R for filtering
- name: Tidyverse
  role: Data-frame manipulation for singleton removal, read-count thresholds, and
    per-library integer subtraction
- name: VSEARCH
  role: Upstream clustering pipeline whose OTU tables are subsequently low-abundance
    filtered
  repo: https://github.com/torognes/vsearch
- name: DADA2
  role: Upstream denoising pipeline whose ASV tables are subsequently low-abundance
    filtered
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:c2c8671fcca0d168928959a4605079797ac78f6de7cbf76fbb80c32d4ed759ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - DADA2
  - Deblur
  - Phyloseq
  - QIIME 2
  - QIIME2R
  - R
  - Tidyverse
  - VSEARCH
  - Vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# feature-table-low-abundance-filtering

## Summary

Applies the paper's standard and extra low-abundance filters to denoised/clustering ASV feature tables: dropping single-sample variants, retaining samples with ≥5,000 filtered reads, and subtracting a per-library fixed integer derived from mock 'miss' sequences to remove unexpected low-abundance ASVs.

## When to use

Use when a denoised or clustered COI metabarcoding feature table (e.g., from DADA2, Deblur, or VSEARCH) still contains singleton or mock-derived spurious ASVs that must be removed before diversity analyses.

## When NOT to use

- Input is already a fully filtered feature table that has had singletons removed and samples rarefied or read-depth thresholded — reapplying would double-filter
- Data are not amplicon ASV/OTU count tables (e.g., shotgun metagenomic gene counts or taxonomy-level tables), since the mock-guided 'miss' subtraction is specific to sequence variants
- No mock-community control exists in the run, making the per-library 'miss' max-count integer impossible to derive

## Inputs

- denoised/clustering ASV feature table (BIOM or TSV) from DADA2, Deblur, or VSEARCH pipelines
- mock-community sample metadata annotating expected, partial, and 'miss' (unexpected) ASV categories
- per-library sample identifiers for the bat guano (PRJNA518082) and mock samples

## Outputs

- standard-filtered feature table (singletons dropped, samples ≥5,000 reads)
- extra-filtered feature table with per-library 'miss'-count subtraction applied, exported in BIOM/TSV format
- verification report confirming zero abundance of all 'miss' mock ASVs and sample read-count compliance

## How to apply

Load the denoised/clustering ASV frequency (feature) table, e.g., with QIIME2R or pandas. Apply the standard filter: drop any sequence variant observed in just one sample across the entire dataset and retain only samples with ≥5,000 total filtered reads. For the extra filter, first determine, per library, the maximum count observed for 'miss' (unexpected) mock sequence variants; then subtract that single fixed integer from every element of that library's feature-table columns, zeroing any resulting negative or zero counts. Verify that all 'miss'-categorized mock ASVs are zeroed and that every retained sample still meets the ≥5,000-read minimum. Rationale: the paper found error-rate differences between denoising and clustering methods are largely mitigated by removing low-abundance sequence variants, and mock communities provide an empirically derived filtering strategy for unexpected sequences.

## Related tools

- **QIIME2R** (Import the QIIME 2-derived feature table into R for filtering)
- **Tidyverse** (Data-frame manipulation for singleton removal, read-count thresholds, and per-library integer subtraction)
- **VSEARCH** (Upstream clustering pipeline whose OTU tables are subsequently low-abundance filtered) — https://github.com/torognes/vsearch
- **DADA2** (Upstream denoising pipeline whose ASV tables are subsequently low-abundance filtered)

## Examples

```
ft_std <- ft[rowSums(ft > 0) > 1, , drop = FALSE]; ft_std <- ft_std[, colSums(ft_std) >= 5000]
```

## Evaluation signals

- Every 'miss'-categorized mock ASV has zero abundance in the extra-filtered output table
- All retained samples meet the ≥5,000 total filtered reads minimum
- No sequence variant in the standard-filtered table is observed in exactly one sample across the entire dataset
- Per-library subtraction values equal the maximum count of 'miss' mock variants in that library, and no counts are negative after subtraction
- Reapplying the filter to already-filtered tables changes no further values (idempotence check)

## Limitations

- The extra filter's fixed-integer subtraction is derived from mock-community 'miss' sequences and is only valid when a well-characterized mock community is processed alongside the biological samples
- The paper reports software versions and pipeline details inconsistently in some sections, so parameter provenance should be confirmed against the tidybug repo scripts
- Filtering choices themselves influenced alpha-diversity estimates (Hill numbers 0, 1, 2), so downstream diversity comparisons must state which filtering regime was applied

## Evidence

- [methods] a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [abstract] these differences are largely mitigated by removing low‐abundance sequence variants: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [intro] Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018): "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
