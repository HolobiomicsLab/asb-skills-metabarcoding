---
name: sequence-alignment-match-classification
description: Use when you have representative sequences from a mock-community metabarcoding
  run and a known list of expected community members, and need to quantify erroneous
  (unexpected) sequences as exact/partial/miss matches to compute error rates per
  denoising or clustering method and filter stringency.
when_to_use_negative:
- When the goal is taxonomic assignment against a large reference database (BOLD,
  GenBank, SILVA) rather than verification against a known mock community composition
  — use a taxonomic classifier instead.
- When input is already a classified feature table with error labels; this skill operates
  on raw representative sequences needing alignment-based verification.
- When no ground-truth list of expected community members exists — exact/partial/miss
  categorization requires known expected sequences.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3299
tools:
- name: VSEARCH
  role: Global aligner used to cluster representative sequences (98%/97% identity)
    and to align mock ASVs against expected references for exact/partial/miss categorization
  repo: https://github.com/torognes/vsearch
- name: DADA2
  role: Denoising program producing ASVs whose unexpected (erroneous) sequences are
    quantified by the match classification
- name: Deblur
  role: Denoising program (run with --p-min-reads 2, --p-min-size 1) producing ASVs
    compared under the same match classification
- name: QIIME 2
  role: Framework (v2018.11) for importing reads, running DADA2/Deblur, and applying
    standard/extra feature-table filters
- name: Cutadapt
  role: Trims raw paired-end reads prior to denoising/clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:345b5848572acc769d21fd32cdcb61bf27748f6683da9eb106fce13f3e4caadc
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# sequence-alignment-match-classification

## Summary

Aligns metabarcoding representative sequences (ASVs/OTUs) from a mock community against a curated set of expected reference sequences, then bins each alignment into 'exact' (100% identity), 'partial' (97–99% identity), or 'miss' (<97% identity) classes using a minimum 97% coverage threshold, enabling computation of per-condition error rates that distinguish denoising (DADA2, Deblur) from clustering (VSEARCH) pipelines.

## When to use

Use when you have representative sequences from a mock-community metabarcoding run and a known list of expected community members, and need to quantify erroneous (unexpected) sequences as exact/partial/miss matches to compute error rates per denoising or clustering method and filter stringency.

## When NOT to use

- When the goal is taxonomic assignment against a large reference database (BOLD, GenBank, SILVA) rather than verification against a known mock community composition — use a taxonomic classifier instead.
- When input is already a classified feature table with error labels; this skill operates on raw representative sequences needing alignment-based verification.
- When no ground-truth list of expected community members exists — exact/partial/miss categorization requires known expected sequences.

## Inputs

- Mock community representative sequence variants (FASTA / QIIME 2 feature-table) from each denoising or clustering program (DADA2, Deblur, VSEARCH)
- Expected mock community reference sequences (curated list of community members)
- Raw paired-end metabarcoding reads (e.g. QIIME 2-imported, Cutadapt-trimmed COI amplicons, unmerged reads ≥175 bp)

## Outputs

- Per-sequence match classification (exact / partial / miss) with identity and coverage values
- mock_error_rates.csv — per-condition error rates for each denoising/clustering program at each filtering level

## How to apply

Run the denoising/clustering programs under comparison — DADA2, Deblur (with --p-min-reads 2 and --p-min-size 1 so only singletons are discarded), and VSEARCH (merge paired reads, dereplicate, cluster at 98% identity, de novo chimera filtering, final 97% clustering) — on the same mock community reads, e.g. QIIME 2 (v2018.11) paired-end COI data trimmed with Cutadapt and unmerged reads retained at a minimum length of 175 bp. Align each mock representative sequence to the expected mock community references, requiring a minimum of 97% coverage, and categorize each match: 'exact' at 100% identity, 'partial' at 97–99% identity, and 'miss' at <97% identity. Apply the basic output plus the standard filter (drop variants observed in only one sample across the dataset; retain samples with ≥5,000 total filtered reads) and, optionally, the extra filter (standard filters plus subtracting the per-library maximum 'miss' variant count from every feature-table element). Compute per-condition error rates for each program at each filtering level and write the summary to mock_error_rates.csv. The rationale is that denoising programs generate error models to address sequence errors, whereas clustering at fixed identity thresholds (95–99%) can inflate the count of unexpected sequences.

## Related tools

- **VSEARCH** (Global aligner used to cluster representative sequences (98%/97% identity) and to align mock ASVs against expected references for exact/partial/miss categorization) — https://github.com/torognes/vsearch
- **DADA2** (Denoising program producing ASVs whose unexpected (erroneous) sequences are quantified by the match classification)
- **Deblur** (Denoising program (run with --p-min-reads 2, --p-min-size 1) producing ASVs compared under the same match classification)
- **QIIME 2** (Framework (v2018.11) for importing reads, running DADA2/Deblur, and applying standard/extra feature-table filters)
- **Cutadapt** (Trims raw paired-end reads prior to denoising/clustering)

## Examples

```
qiime demux import + qiime dada2 denoise-paired / deblur denoise-other / vsearch cluster-features at 97%; then align mock ASVs to expected references and write mock_error_rates.csv
```

## Evaluation signals

- The alignment output partitions every mock representative sequence into exactly one of 'exact' (100% identity), 'partial' (97–99%), or 'miss' (<97%), each meeting ≥97% coverage.
- mock_error_rates.csv shows denoising methods (DADA2, Deblur) have lower error rates than VSEARCH clustering under the basic filter — e.g. partial: 9 (DADA2) vs 146 (Deblur) vs 295 (VSEARCH); miss: 40 vs 576 vs 753.
- After applying the standard filter (drop single-sample variants; retain samples with ≥5,000 filtered reads) and the extra filter, differences in unexpected ASV counts between methods are largely mitigated.
- Chimeric and low-abundance artifacts are controlled: Deblur uses --p-min-reads 2 / --p-min-size 1, and VSEARCH applies de novo chimera filtering before final 97% clustering.

## Limitations

- Error-rate differences between denoising and clustering are largely mitigated by removing low-abundance sequence variants, so results depend strongly on the chosen filtering stringency (basic vs standard vs extra).
- The evaluation is grounded in mock communities and COI amplicon data (181 bp fragment, QIIME 2 v2018.11); thresholds such as 97% identity and 97% coverage may not transfer directly to other markers or markers with higher sequencing error.
- Clustering radius choices (95%, 97%, 99%) affect reference composition and downstream comparisons, so cross-method conclusions are sensitive to clustering parameters.

## Evidence

- [abstract] Denoising methods have lower error rates than traditional clustering methods, largely mitigated by removing low-abundance sequence variants: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [intro] Mock communities provide an empirically derived filtering strategy to assess unexpected sequences: "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
- [methods] Standard filter definition: drop single-sample variants, retain samples with >=5,000 filtered reads: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] Clustering at 99%, 97%, and 95% identity is part of the pipeline: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [intro] Denoising programs like DADA2 and Deblur generate error models to address sequence errors: "Denoising programs like DADA2 (Callahan et al., 2016) or Deblur (Amir et al., 2017) generate error models to address potential sequence errors"
