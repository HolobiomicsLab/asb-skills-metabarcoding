---
name: fastq-sequence-concatenation
description: Use when paired-end amplicon reads have been merged per sample (e.g.
  temp/Sample1.merged.fq) and a uniform filtering pass (primer strip + max expected
  error) must be applied across all samples in one vsearch call before OTU/ASV analysis.
when_to_use_negative:
- Input is per-sample FASTQ that has not yet been merged and relabeled with sample-ID
  prefixes — merging (--fastq_mergepairs) must come first so headers carry sample
  provenance.
- Input is already a pooled/concatenated FASTQ awaiting only OTU clustering, dereplication,
  or chimera removal — this concatenation step belongs strictly in the preprocessing
  stage before primer trimming.
- Samples have incompatible primer designs or need sample-specific trimming thresholds
  — a single concatenated filter pass would apply one uniform 29 bp/18 bp strip and
  maxee rate to all reads.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Merges each sample's paired-end reads (--fastq_mergepairs, --relabel with
    sample-ID prefix) into the per-sample merged FASTQ files that are subsequently
    concatenated; its --fastx_filter step consumes the concatenated pool
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: 'Pipeline that defines the concatenation workflow: per-sample merged FASTQs
    in temp/ are pooled with ''cat'' into temp/all.fq before primer trimming and quality
    filtering (sections 6.2.2–6.2.3)'
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:e56025dc374aaa08f5d31dd24037959b04aa87b1d9b1e1c32da0e8f460f5a708
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fastq-sequence-concatenation

## Summary

Pooling per-sample merged amplicon FASTQ files into a single concatenated file (temp/all.fq) so subsequent primer-trimming and quality filtering can be run once over all samples, as done in the EasyAmplicon preprocessing module (sections 6.2.2–6.2.3).

## When to use

Use when paired-end amplicon reads have been merged per sample (e.g. temp/Sample1.merged.fq) and a uniform filtering pass (primer strip + max expected error) must be applied across all samples in one vsearch call before OTU/ASV analysis.

## When NOT to use

- Input is per-sample FASTQ that has not yet been merged and relabeled with sample-ID prefixes — merging (--fastq_mergepairs) must come first so headers carry sample provenance.
- Input is already a pooled/concatenated FASTQ awaiting only OTU clustering, dereplication, or chimera removal — this concatenation step belongs strictly in the preprocessing stage before primer trimming.
- Samples have incompatible primer designs or need sample-specific trimming thresholds — a single concatenated filter pass would apply one uniform 29 bp/18 bp strip and maxee rate to all reads.

## Inputs

- Per-sample merged FASTQ files produced by 'vsearch --fastq_mergepairs' (temp/*.merged.fq, e.g. temp/Sample1.merged.fq) with reads relabeled by sample-ID prefix

## Outputs

- A single pooled FASTQ file combining all samples (temp/all.fq), which is then fed to 'vsearch --fastx_filter' to produce the quality-filtered amplicon FASTA temp/filtered.fa

## How to apply

After each sample's paired-end reads are merged with 'vsearch --fastq_mergepairs' and relabeled with a sample-ID prefix (e.g. --relabel Sample1.), gather all per-sample merged FASTQ files by concatenating them: 'cat temp/*.merged.fq > temp/all.fq'. This pooled file is the single input to the downstream 'vsearch --fastx_filter' step, which strips 29 bp from the left and 18 bp from the right (the primers) with '--fastq_maxee_rate 0.01' and writes 'temp/filtered.fa'. Concatenating before filtering guarantees identical trim and expected-error thresholds for every sample and preserves sample provenance because read headers already carry the sample-ID prefix from the relabeling step. Validate by confirming temp/all.fq contains records from every merged per-sample file (e.g. count records per sample prefix with grep) before running the filter.

## Related tools

- **VSEARCH** (Merges each sample's paired-end reads (--fastq_mergepairs, --relabel with sample-ID prefix) into the per-sample merged FASTQ files that are subsequently concatenated; its --fastx_filter step consumes the concatenated pool) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline that defines the concatenation workflow: per-sample merged FASTQs in temp/ are pooled with 'cat' into temp/all.fq before primer trimming and quality filtering (sections 6.2.2–6.2.3)) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
cat temp/*.merged.fq > temp/all.fq && vsearch --fastx_filter temp/all.fq --fastq_stripleft 29 --fastq_stripright 18 --fastq_maxee_rate 0.01 --fastaout temp/filtered.fa
```

## Evaluation signals

- temp/all.fq exists and is non-empty after 'cat temp/*.merged.fq > temp/all.fq'.
- Every sample is represented in the pool: read headers carry each sample-ID prefix set during --relabel, so counting records per prefix (e.g. grep -c '^@Sample1.') matches the per-sample merged file record counts.
- Total record count in temp/all.fq equals the sum of record counts across all per-sample temp/*.merged.fq inputs.
- Downstream 'vsearch --fastx_filter' run on temp/all.fq with --fastq_stripleft 29 --fastq_stripright 18 --fastq_maxee_rate 0.01 completes and writes temp/filtered.fa.
- temp/filtered.fa begins with well-formed FASTA records whose first bases correspond to sequence downstream of the 29 bp forward primer, verifiable with 'head'.

## Limitations

- The concatenation is only valid if per-sample merged files were relabeled with sample-ID prefixes (--relabel Sample1.); otherwise sample identity is lost when files are pooled.
- Downstream filtering (29 bp left / 18 bp right strip, --fastq_maxee_rate 0.01) assumes a single uniform primer design for all pooled samples; mixed amplicon assays cannot share one concatenated filter pass.
- No quality trimming occurs at the concatenation step itself — reads failing the 1% expected-error threshold are only dropped later in --fastx_filter.

## Evidence

- [readme] VSEARCH can directly read input query and database files that are compressed using gzip (.gz) and bzip2 (.bz2): [evidence span withheld — anchor status: external_doc]
- [readme] It also supports FASTQ file analysis, filtering, conversion and merging of paired-end reads.: [evidence span withheld — anchor status: external_doc]
- [abstract] including data preprocessing (quality filtering, chimera removal): "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis"
- [intro] These tools work together to assess sequence quality, split barcoded sequences, remove sequencing background: "These tools work together to assess sequence quality, split barcoded sequences, remove sequencing background, and identify amplicon sequence variants (ASVs) and operational taxonomic units (OTUs)."
