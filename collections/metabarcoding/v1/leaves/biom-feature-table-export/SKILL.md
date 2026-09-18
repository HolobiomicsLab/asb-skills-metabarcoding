---
name: biom-feature-table-export
description: Use when a denoised (DADA2/Deblur) or clustered (99/97/95%) mock-community
  COI feature table has been filtered and must be exported in BIOM/TSV format with
  verified removal of unexpected mock ASVs.
when_to_use_negative:
- The dataset has no mock community samples, so per-library maximum 'miss' counts
  cannot be empirically derived for the extra filter
- The input is already a fully filtered feature table (re-applying the extra subtraction
  would incorrectly zero legitimate low-abundance real taxa)
- The goal is unfiltered comparison of raw denoising vs clustering outputs, where
  the paper notes error-rate differences are only mitigated by low-abundance removal
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: R
  role: Scripting environment in which the standard and extra filters are applied
    to the mock ASV table before export
- name: QIIME2R
  role: R package used to load the denoised/clustering feature table (.qza) prior
    to filtering and BIOM/TSV export
- name: Tidyverse
  role: R packages used for per-sample filtering, per-library subtraction of the fixed
    integer, and writing the filtered table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:75ecc61cd80ea49e584963599bbc1a21fd1eb6a2f4dc53dedd087178a1ad6c54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2R
  - R
  - Tidyverse
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 4
---

# biom-feature-table-export

## Summary

Export the filtered mock-community ASV/OTU feature table (after the standard and extra mock-guided filters) as a canonical BIOM/TSV file, using R/QIIME2R or pandas, so that downstream diversity and classifier comparisons operate on a verified table with no unexpected 'miss' variants.

## When to use

Use when a denoised (DADA2/Deblur) or clustered (99/97/95%) mock-community COI feature table has been filtered and must be exported in BIOM/TSV format with verified removal of unexpected mock ASVs.

## When NOT to use

- The dataset has no mock community samples, so per-library maximum 'miss' counts cannot be empirically derived for the extra filter
- The input is already a fully filtered feature table (re-applying the extra subtraction would incorrectly zero legitimate low-abundance real taxa)
- The goal is unfiltered comparison of raw denoising vs clustering outputs, where the paper notes error-rate differences are only mitigated by low-abundance removal

## Inputs

- denoised or clustered mock-community ASV/OTU frequency table (QIIME2 .qza artifact, BIOM, or TSV)
- per-ASV mock-community classification labels distinguishing expected 'mock' vs unexpected 'miss' variants
- per-library metadata identifying which samples belong to each mock community library

## Outputs

- filtered feature table exported in BIOM format
- filtered feature table exported in TSV format
- validation summary showing zero abundance for all 'miss' ASVs and >=5,000 reads per retained sample

## How to apply

Load the denoised/clustering mock-community ASV frequency table from the prior task using QIIME2R (read_qza) or pandas. Apply the standard filter: drop any sequence variant observed in just one sample across the entire dataset and retain only samples with >=5,000 total filtered reads. Then apply the extra filter: for each library, determine the maximum count of 'miss' (unexpected) mock variants and subtract that fixed integer from every element of the feature table, zeroing negative or zero counts. Confirm every 'miss'-categorized mock ASV is absent, then export the resulting table in BIOM or TSV format as the canonical output for downstream diversity metric and classifier comparisons.

## Related tools

- **R** (Scripting environment in which the standard and extra filters are applied to the mock ASV table before export)
- **QIIME2R** (R package used to load the denoised/clustering feature table (.qza) prior to filtering and BIOM/TSV export)
- **Tidyverse** (R packages used for per-sample filtering, per-library subtraction of the fixed integer, and writing the filtered table)

## Examples

```
Rscript -e 'library(qiime2R); tab <- read_qza("mock_table.qza")$data; tab[tab <= missmax] <- 0; write.table(tab, "filtered_table.tsv", sep="\t", quote=FALSE)'
```

## Evaluation signals

- Every 'miss'-categorized mock ASV has zero abundance in the exported table
- All retained samples meet the >=5,000 total filtered reads minimum
- No sequence variant in the output is observed in only one sample across the dataset (standard filter invariant)
- Each per-library subtraction used the maximum count observed for 'miss' variants in that library's mock sample, so no 'miss' variant retains a positive count
- Exported file parses as a valid BIOM/TSV feature table with sample x feature dimensions matching the filtered sample set

## Limitations

- The extra filter subtracts a single fixed integer per library from every feature-table element, which can remove genuine low-abundance sequences of real taxa, not just 'miss' variants
- The strategy is empirically derived from mock communities; without a mock community the per-library 'miss' maximum cannot be computed
- The paper explicitly avoids recommending a single best pathway: filtering choices alter diversity metric results, so exported tables are only comparable within a matched filtering regime

## Evidence

- [methods] Standard filter threshold: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] Extra filter definition: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [intro] Mock-guided filtering rationale: "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
- [readme] Repo scope: "Methods and data describing filtering COI datasets and database construction."
