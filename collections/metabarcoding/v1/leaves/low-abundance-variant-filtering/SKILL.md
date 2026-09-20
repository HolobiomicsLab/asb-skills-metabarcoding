---
name: low-abundance-variant-filtering
description: Use when a QIIME 2 feature table generated from mock-community or diet
  metabarcoding reads contains unexpected ASVs and you need to assess how standard/extra
  low-abundance filtering changes per-method error rates.
when_to_use_negative:
- Input is already a heavily filtered feature table where singletons and low-read
  samples have been removed — reapplying the standard filter changes nothing and the
  extra subtraction could distort true counts
- The study goal is to compare denoising programs' raw error profiles (e.g., unexpected
  ASV counts) without mitigation, since this skill's purpose is precisely to remove
  that signal
- Working with non-metabarcoding data lacking an expected-species reference, since
  the 'miss'-count subtraction in the extra filter is derived from mock community
  alignments
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: QIIME 2
  role: Framework (v2018.11) in which feature-table filtering and the Deblur denoising
    with singleton-discard parameters are executed
  repo: github.com/devonorourke/tidybug
- name: DADA2
  role: Denoising program producing the low-error-rate baseline against which filtered
    vs unfiltered unexpected-ASV counts are compared
  repo: github.com/devonorourke/tidybug
- name: Deblur
  role: Denoising program run with --p-min-reads 2 and --p-min-size 1 so only singleton
    reads are discarded
  repo: github.com/devonorourke/tidybug
- name: VSEARCH
  role: Clustering program (98% then 97% identity with de novo chimera filtering)
    whose excess unexpected ASVs motivate the low-abundance filtering
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:9e9e606b5bf0c48ae64f3ff2ad375f4f13828160d41708a8dd06b0866e7dbd32
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cutadapt
  - DADA2
  - Deblur
  - QIIME 2
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# low-abundance-variant-filtering

## Summary

A post-denoising/clustering filtering strategy for COI metabarcoding feature tables in which low-abundance sequence variants (singletons and variants observed in few samples) are removed to mitigate the excess of unexpected (erroneous) ASVs produced by clustering methods like VSEARCH relative to denoisers like DADA2 and Deblur.

## When to use

Use when a QIIME 2 feature table generated from mock-community or diet metabarcoding reads contains unexpected ASVs and you need to assess how standard/extra low-abundance filtering changes per-method error rates.

## When NOT to use

- Input is already a heavily filtered feature table where singletons and low-read samples have been removed — reapplying the standard filter changes nothing and the extra subtraction could distort true counts
- The study goal is to compare denoising programs' raw error profiles (e.g., unexpected ASV counts) without mitigation, since this skill's purpose is precisely to remove that signal
- Working with non-metabarcoding data lacking an expected-species reference, since the 'miss'-count subtraction in the extra filter is derived from mock community alignments

## Inputs

- QIIME 2 feature table (and representative-sequence artifacts) from DADA2-, Deblur-, or VSEARCH-processed mock community paired-end COI reads (trimmed with Cutadapt, unmerged reads >=175 bp retained)
- Expected mock community reference sequences for categorizing matches as 'exact', 'partial', or 'miss'

## Outputs

- Filtered feature tables (standard-filtered and extra-filtered variants)
- mock_error_rates.csv — per-condition error rates for each denoising/clustering program at each filtering level

## How to apply

After generating representative sequence variants (DADA2, Deblur with --p-min-reads 2 and --p-min-size 1 so only singletons are discarded, or VSEARCH clustering at 98% then 97% identity), apply a 'standard filter' that (a) drops any sequence variant observed in just one sample across the entire dataset and (b) retains only samples with >=5,000 total filtered reads. Optionally apply an 'extra filter' that incorporates the standard filters and subtracts a single, fixed integer — the per-library maximum 'miss' variant count — from every element of the feature table. Rationale: the paper found denoising methods have lower error rates than clustering (DADA2: 9 partial / 40 miss unexpected mock ASVs vs Deblur 146/576 and VSEARCH 295/753 under basic filtering), but these differences are largely mitigated by removing low-abundance variants. Compare per-condition error rates across filtering levels by aligning each mock representative sequence to expected mock references (exact = 100% identity, partial = 97–99%, miss = <97%, minimum 97% coverage) and writing results to mock_error_rates.csv.

## Related tools

- **QIIME 2** (Framework (v2018.11) in which feature-table filtering and the Deblur denoising with singleton-discard parameters are executed) — github.com/devonorourke/tidybug
- **DADA2** (Denoising program producing the low-error-rate baseline against which filtered vs unfiltered unexpected-ASV counts are compared) — github.com/devonorourke/tidybug
- **Deblur** (Denoising program run with --p-min-reads 2 and --p-min-size 1 so only singleton reads are discarded) — github.com/devonorourke/tidybug
- **VSEARCH** (Clustering program (98% then 97% identity with de novo chimera filtering) whose excess unexpected ASVs motivate the low-abundance filtering) — https://github.com/torognes/vsearch

## Examples

```
qiime deblur denoise-other --i-demultiplexed-seqs demux.qza --p-min-reads 2 --p-min-size 1 --o-table deblur_table.qza
```

## Evaluation signals

- mock_error_rates.csv shows DADA2 with fewer unexpected mock ASVs than Deblur and VSEARCH under basic filtering (partial: 9 vs 146 and 295; miss: 40 vs 576 and 753)
- After applying the standard/extra filter, per-method error-rate differences are largely mitigated, consistent with the manuscript's reported finding
- Every feature-table element in the extra-filtered table equals the standard-filtered value minus the per-library maximum 'miss' variant count
- No sequence variant observed in only one sample remains, and no retained sample has fewer than 5,000 filtered reads

## Limitations

- The filtering strategy is empirically derived from a mock community; transferability to real diet samples (e.g., bat guano) is an assumption, and the paper notes mock communities 'can provide an empirically derived filtering strategy'
- Software versions for the denoising programs and classifiers are not reported in the results section, limiting exact reproducibility
- The extra filter's fixed-integer subtraction assumes the per-library maximum 'miss' count is a defensible estimate of spurious abundance; it is a heuristic rather than a validated error model

## Evidence

- [abstract] denoising lower error rates mitigated: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [methods] standard filter definition: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] extra filter definition: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [intro] mock-community-guided filtering rationale: [evidence span withheld — anchor status: not_found]
- [readme] project docs pointer: "See [the docs folder of this repo](https://github.com/devonorourke/tidybug/tree/master/docs) for information on how each part of the project was performed"
