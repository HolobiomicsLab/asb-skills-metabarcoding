---
name: qiime2-artifact-processing
description: Use when demultiplexed paired-end amplicon reads (e.g., COI from mock
  communities or bat guano) must be imported to QIIME 2, trimmed, denoised or clustered
  into feature tables, and compared for unexpected-sequence error rates under alternative
  filtering stringencies.
when_to_use_negative:
- Input is already a feature table of representative sequences — no denoising/clustering
  step is needed; go directly to filtering or diversity analyses
- The goal is taxonomic classification of representative sequences or reference database
  construction (BOLD/GenBank COI), which are separate workflows in this project
- The dataset has no expected mock community composition, making the exact/partial/miss
  error-rate benchmarking meaningless
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: QIIME 2
  role: Platform (v2018.11) for importing reads, running Cutadapt trimming, DADA2/Deblur
    denoising, and filtering feature tables
- name: Cutadapt
  role: Trims raw mock community paired-end reads before denoising, retaining unmerged
    reads ≥175 bp
- name: DADA2
  role: Denoising program generating error models to produce representative sequence
    variants for comparison against clustering
- name: Deblur
  role: Denoising program run with --p-min-reads 2 and --p-min-size 1 so only singleton
    reads are discarded
- name: VSEARCH
  role: 'Clustering alternative: merges paired reads, dereplicates, clusters at 98%
    identity, de novo chimera filtering, then final clustering at 97% identity'
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:cb882ad7382dbcf8119cc30825cb5e64f82405402951ca7d97f32e22d24f17d3
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# qiime2-artifact-processing

## Summary

A QIIME 2 (v2018.11) artifact-processing workflow for COI metabarcoding that turns demultiplexed paired-end reads into denoised (DADA2, Deblur) or clustered (VSEARCH) feature tables, applies low-abundance filters, and quantifies unexpected mock-community ASVs to benchmark error rates.

## When to use

Use when demultiplexed paired-end amplicon reads (e.g., COI from mock communities or bat guano) must be imported to QIIME 2, trimmed, denoised or clustered into feature tables, and compared for unexpected-sequence error rates under alternative filtering stringencies.

## When NOT to use

- Input is already a feature table of representative sequences — no denoising/clustering step is needed; go directly to filtering or diversity analyses
- The goal is taxonomic classification of representative sequences or reference database construction (BOLD/GenBank COI), which are separate workflows in this project
- The dataset has no expected mock community composition, making the exact/partial/miss error-rate benchmarking meaningless

## Inputs

- Demultiplexed paired-end FASTQ reads (COI metabarcoding; mock community and bat guano samples, NCBI SRA PRJNA518082)
- Expected mock community reference sequences (for exact/partial/miss alignment)
- QIIME 2 artifacts (.qza) for feature tables and representative sequences produced by each denoising/clustering program

## Outputs

- Feature tables and representative-sequence artifacts per method (DADA2, Deblur, VSEARCH) per filtering level (basic, standard, extra)
- Mock ASV match categorization: 'exact' (100% identity), 'partial' (97–99%), 'miss' (<97%) at ≥97% coverage
- Per-condition error-rate summary written to mock_error_rates.csv

## How to apply

Import raw mock community paired-end reads into QIIME 2 (v2018.11) and trim with Cutadapt, retaining unmerged reads with a minimum length of 175 bp. Generate representative sequence variants three ways: DADA2 denoising, Deblur denoising with --p-min-reads 2 and --p-min-size 1 (so only singleton reads are discarded), and VSEARCH by merging paired reads, dereplicating, clustering at 98% identity, de novo chimera filtering, and final clustering at 97% identity. Apply a standard filter that (a) drops any sequence variant observed in just one sample across the dataset and (b) retains only samples with ≥5,000 total filtered reads, plus an extra filter that further subtracts the per-library maximum 'miss' variant count from each feature-table element. Align each mock representative sequence to the expected mock community references with ≥97% coverage, categorizing matches as 'exact' (100% identity), 'partial' (97–99%), or 'miss' (<97%). Compute per-condition error rates per denoising/clustering program at each filtering level and write them to mock_error_rates.csv; the rationale is that denoising methods have lower error rates than VSEARCH clustering under the basic filter, but these differences are largely mitigated by removing low-abundance sequence variants.

## Related tools

- **QIIME 2** (Platform (v2018.11) for importing reads, running Cutadapt trimming, DADA2/Deblur denoising, and filtering feature tables)
- **Cutadapt** (Trims raw mock community paired-end reads before denoising, retaining unmerged reads ≥175 bp)
- **DADA2** (Denoising program generating error models to produce representative sequence variants for comparison against clustering)
- **Deblur** (Denoising program run with --p-min-reads 2 and --p-min-size 1 so only singleton reads are discarded)
- **VSEARCH** (Clustering alternative: merges paired reads, dereplicates, clusters at 98% identity, de novo chimera filtering, then final clustering at 97% identity) — https://github.com/torognes/vsearch

## Examples

```
qiime deblur denoise-16S --i-demultiplexed-seqs demux.qza --p-min-reads 2 --p-min-size 1 --output-dir deblur_out
```

## Evaluation signals

- Mock representative sequences align to expected references at ≥97% coverage and fall into exact (100%), partial (97–99%), or miss (<97%) categories
- Under the basic filter, DADA2 yields fewer unexpected mock ASVs than Deblur and VSEARCH (partial: 9 vs 146 vs 295; miss: 40 vs 576 vs 753)
- Standard filter output shows single-sample variants dropped and samples retained only with ≥5,000 total filtered reads
- After standard/extra low-abundance filtering, error-rate differences between denoising and clustering methods are largely mitigated, consistent with the reported finding
- mock_error_rates.csv is produced with per-condition error rates for each program at each filtering level

## Limitations

- Software versions for the denoising programs and classifiers are not reported in the results section, complicating exact reproduction
- The extra filter subtracts a fixed integer (per-library maximum 'miss' variant count) from every feature-table element, a mock-guided adjustment that is not generalizable to datasets without expected community composition
- Deblur settings (--p-min-reads 2, --p-min-size 1) mean only singleton reads are discarded, so method comparisons are sensitive to these abundance-threshold choices and to the 175 bp minimum length for unmerged reads

## Evidence

- [intro] Denoising programs like DADA2 (Callahan et al., 2016) or Deblur (Amir et al., 2017) generate error models to address potential sequence errors: "Denoising programs like DADA2 (Callahan et al., 2016) or Deblur (Amir et al., 2017) generate error models to address potential sequence errors"
- [methods] standard filter definition: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] extra filter definition: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [abstract] error-rate finding: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [readme] repo documentation pointer: "Methods and data describing filtering COI datasets and database construction."
