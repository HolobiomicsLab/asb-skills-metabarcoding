---
name: mock-community-error-rate-quantification
description: Use when you have raw paired-end mock community reads and need to compare
  how representative-sequence inference methods (denoising vs OTU clustering) and
  filtering stringency affect erroneous-sequence rates.
when_to_use_negative:
- Input is already a feature table of classified taxa rather than raw mock community
  reads — this skill requires re-running representative-sequence inference from the
  raw data.
- The biological dataset has no known expected community composition, so 'unexpected'
  sequences cannot be defined by comparison to references.
- Your goal is taxonomic classifier accuracy (e.g. BOLD API vs BLAST+LCA comparison)
  rather than denoising/clustering error assessment.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QIIME 2
  role: Framework for importing mock reads and running DADA2/Deblur denoising and
    feature-table filtering
- name: DADA2
  role: Error-model denoising to generate ASVs from the mock community reads
- name: Deblur
  role: Alternative denoising program run with --p-min-reads 2 and --p-min-size 1
    for comparison against DADA2
- name: VSEARCH
  role: Clustering-based OTU pipeline (merge, dereplicate, 98%/97% identity clustering,
    de novo chimera filtering) as the clustering comparison arm
  repo: https://github.com/torognes/vsearch
- name: Cutadapt
  role: Trims primers from raw mock community paired-end reads before inference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:44f45493179fc362efb1b1f7a3f8003aff5037b868b76268f2962664d373d94a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - Cutadapt
  - DADA2
  - Deblur
  - QIIME 2
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mock-community-error-rate-quantification

## Summary

Quantifies unexpected (erroneous) ASVs in a mock community metabarcoding dataset by aligning representative sequences from each denoising or clustering program (DADA2, Deblur, VSEARCH) against the known expected mock references, then computing per-condition error rates under escalating low-abundance filtering.

## When to use

Use when you have raw paired-end mock community reads and need to compare how representative-sequence inference methods (denoising vs OTU clustering) and filtering stringency affect erroneous-sequence rates.

## When NOT to use

- Input is already a feature table of classified taxa rather than raw mock community reads — this skill requires re-running representative-sequence inference from the raw data.
- The biological dataset has no known expected community composition, so 'unexpected' sequences cannot be defined by comparison to references.
- Your goal is taxonomic classifier accuracy (e.g. BOLD API vs BLAST+LCA comparison) rather than denoising/clustering error assessment.

## Inputs

- Raw mock community paired-end amplicon reads (FASTQ, 181 bp COI fragment, dual-indexed)
- Expected mock community reference sequences (FASTA) for exact/partial/miss classification
- Feature tables and representative-sequence FASTAs from each representative-sequence inference program (DADA2, Deblur, VSEARCH)

## Outputs

- mock_error_rates.csv with per-condition error rates for each denoising/clustering program at each filtering level (basic, standard, extra)
- Categorized mock ASV counts (exact / partial / miss) per program and filter level
- Feature tables after standard and extra low-abundance filtering

## How to apply

Import raw mock community paired-end reads into QIIME 2 (v2018.11), trim primers with Cutadapt, and retain unmerged reads with a minimum length of 175 bp. Generate representative sequence variants in parallel: DADA2 and Deblur (--p-min-reads 2, --p-min-size 1 so only singletons are discarded) for denoising, and VSEARCH by merging paired reads, dereplicating, de novo chimera filtering, and clustering at 98% then 97% identity. Align each mock representative sequence to the expected mock community references with a minimum of 97% coverage, categorizing matches as 'exact' (100% identity), 'partial' (97-99%), or 'miss' (<97%). Apply the basic output plus a standard filter (drop sequence variants observed in only one sample; retain samples with ≥5,000 filtered reads) and an extra filter (standard filters plus subtracting the per-library maximum 'miss' variant count from every feature-table element). Compute per-condition error rates per program at each filtering level and write mock_error_rates.csv; the table should show denoising yields fewer unexpected ASVs (e.g. partial: 9 for DADA2 vs 146 Deblur and 295 VSEARCH; miss: 40 vs 576 and 753) under basic filtering, with differences largely mitigated after standard/extra filtering.

## Related tools

- **QIIME 2** (Framework for importing mock reads and running DADA2/Deblur denoising and feature-table filtering)
- **DADA2** (Error-model denoising to generate ASVs from the mock community reads)
- **Deblur** (Alternative denoising program run with --p-min-reads 2 and --p-min-size 1 for comparison against DADA2)
- **VSEARCH** (Clustering-based OTU pipeline (merge, dereplicate, 98%/97% identity clustering, de novo chimera filtering) as the clustering comparison arm) — https://github.com/torognes/vsearch
- **Cutadapt** (Trims primers from raw mock community paired-end reads before inference)

## Examples

```
./vsearch --usearch_global queries.fsa --db database.fsa --id 0.9 --alnout alnout.txt
```

## Evaluation signals

- mock_error_rates.csv is produced with one error-rate value per denoising/clustering program per filtering level (basic, standard, extra).
- Under the basic filter, denoising methods (DADA2, Deblur) show fewer unexpected mock ASVs than VSEARCH clustering (e.g. partial: 9 vs 146 vs 295; miss: 40 vs 576 vs 753).
- After standard/extra low-abundance filtering, the error-rate gap between denoising and clustering methods is substantially reduced, consistent with the manuscript's finding.
- Every mock representative sequence aligns to expected references with at least 97% coverage and is binned into exactly one of exact (100%), partial (97-99%), or miss (<97%).
- Standard filter invariants hold: no sequence variant observed in just one sample remains, and all retained samples have ≥5,000 total filtered reads.

## Limitations

- Under the basic (unfiltered) output, differences between denoising and clustering methods are large; conclusions drawn without low-abundance filtering will overstate the method effect.
- Software versions for the denoising programs are not reported in the results section of the manuscript (only QIIME 2 v2018.11 is specified in the workflow).
- The 'extra' filter's fixed subtraction uses the per-library maximum 'miss' variant count, which assumes the mock community composition is fully known and may not transfer to non-mock datasets.

## Evidence

- [abstract] denoising methods have lower error rates than clustering methods, largely mitigated by removing low-abundance sequence variants: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [methods] standard filter definition: drop singletons observed in one sample, retain samples with ≥5,000 filtered reads: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] extra filter definition: standard filters plus subtracting a fixed integer from each feature-table element: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
