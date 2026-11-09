---
name: amplicon-read-qc
description: Use when raw paired-end amplicon FASTQ files (e.g. seq/Sample1_1.fq.gz
  + Sample1_2.fq.gz) must be merged, primer-trimmed, and quality-filtered to an expected-error
  threshold before dereplication, OTU/ASV inference, or chimera removal in EasyAmplicon.
when_to_use_negative:
- Input is already a dereplicated FASTA or an OTU/ASV feature table — read-level merging
  and trimming would be meaningless at this stage
- Reads come from PacBio/Nanopore long amplicon sequencing, which EasyAmplicon 2 routes
  through a different pipeline rather than this paired-end merge step
- Primer sequences vary per sample such that fixed --fastq_stripleft/--fastq_stripright
  lengths would leave primer bases or cut into inserts; use the cutadapt route instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Performs the paired-end merging (--fastq_mergepairs), primer stripping, and
    expected-error filtering (--fastx_filter) that generate temp/filtered.fa
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline that scripts the merge/trim/filter preprocessing step (sections 6.2.2–6.2.3)
    and supplies per-sample naming and validation conventions
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: cutadapt
  role: Alternative primer-removal tool when actual primer sequences must be matched
    (--discard-untrimmed) instead of fixed-length stripping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:ce8858f72c561d656628539a7b753c58032b4ca325330ab1a5bf1d15a07f1d4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - VSEARCH
  - cutadapt
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# amplicon-read-qc

## Summary

Produces clean amplicon sequences (temp/filtered.fa) from raw paired-end Illumina/BGI FASTQ by merging read pairs per sample, stripping primer bases, and applying an expected-error quality filter with VSEARCH as orchestrated in the EasyAmplicon preprocessing module.

## When to use

Use when raw paired-end amplicon FASTQ files (e.g. seq/Sample1_1.fq.gz + Sample1_2.fq.gz) must be merged, primer-trimmed, and quality-filtered to an expected-error threshold before dereplication, OTU/ASV inference, or chimera removal in EasyAmplicon.

## When NOT to use

- Input is already a dereplicated FASTA or an OTU/ASV feature table — read-level merging and trimming would be meaningless at this stage
- Reads come from PacBio/Nanopore long amplicon sequencing, which EasyAmplicon 2 routes through a different pipeline rather than this paired-end merge step
- Primer sequences vary per sample such that fixed --fastq_stripleft/--fastq_stripright lengths would leave primer bases or cut into inserts; use the cutadapt route instead

## Inputs

- Per-sample gzipped paired-end FASTQ files (e.g. seq/Sample1_1.fq.gz forward, seq/Sample1_2.fq.gz reverse) downloaded from GSA CRA002352 in the tutorial
- Known primer lengths (29 bp forward, 18 bp reverse) for the amplicon target

## Outputs

- temp/Sample1.merged.fq — per-sample merged FASTQ with reads relabeled by sample ID prefix
- temp/all.fq — concatenated merged FASTQ across all samples
- temp/filtered.fa — quality-filtered, primer-trimmed amplicons in FASTA format

## How to apply

For each sample, merge the read pairs with `vsearch --fastq_mergepairs` (forward vs --reverse FASTQ), using `--relabel Sample1.` so every merged read carries its sample ID prefix, writing temp/Sample1.merged.fq. Concatenate all per-sample merged files with `cat temp/*.merged.fq > temp/all.fq`. Then run `vsearch --fastx_filter` on temp/all.fq with `--fastq_stripleft 29` and `--fastq_stripright 18` to remove the 29 bp forward and 18 bp reverse primer bases, and `--fastq_maxee_rate 0.01` to drop any read whose expected errors exceed 1% of read length, writing `--fastaout temp/filtered.fa`. The rate-based maxee (rather than a fixed --fastq_maxee) keeps the expected-error criterion proportional to read length, which the article's methods section treats as a variant of its length/quality filtering step. When primer sequences rather than fixed lengths vary, the article describes a cutadapt alternative (-g with both primer sequences and --discard-untrimmed), and length bounds (e.g. --fastq_minlen 1200 --fastq_maxlen 1800) can be added to --fastx_filter for long-amplicon targets.

## Related tools

- **VSEARCH** (Performs the paired-end merging (--fastq_mergepairs), primer stripping, and expected-error filtering (--fastx_filter) that generate temp/filtered.fa) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline that scripts the merge/trim/filter preprocessing step (sections 6.2.2–6.2.3) and supplies per-sample naming and validation conventions) — https://github.com/YongxinLiu/EasyAmplicon
- **cutadapt** (Alternative primer-removal tool when actual primer sequences must be matched (--discard-untrimmed) instead of fixed-length stripping)

## Examples

```
vsearch --fastx_filter temp/all.fq --fastq_stripleft 29 --fastq_stripright 18 --fastq_maxee_rate 0.01 --fastaout temp/filtered.fa
```

## Evaluation signals

- temp/filtered.fa exists and `head` shows correctly formatted FASTA records whose sequences begin after the primer region
- Read length in temp/filtered.fa equals the merged overlap length minus 47 bp (29 + 18 stripped primer bases)
- Sample ID prefixes from --relabel (e.g. 'Sample1.') are preserved in record headers so reads remain traceable to samples
- Filtered read counts are consistent with reads dropped for expected errors > 1% of read length (--fastq_maxee_rate 0.01); no read with maxee_rate above threshold survives
- temp/all.fq contains the concatenation of all per-sample temp/*.merged.fq files (line-count check)

## Limitations

- Fixed --fastq_stripleft 29 / --fastq_stripright 18 only work for amplicons with known, constant primer lengths; sequence-specific trimming requires cutadapt with --discard-untrimmed
- The step removes primers and low-expected-error reads but not chimeras — chimera removal is a separate downstream step (de novo cluster_otus or --uchime_ref against e.g. rdp_16s_v18.fa)
- The article presents the rate-based expected error (--fastq_maxee_rate 0.01) as a per-read-length criterion; for variable-length targets an absolute --fastq_maxee plus length bounds (--fastq_minlen/--fastq_maxlen, e.g. 1200–1800) may be more appropriate

## Evidence

- [methods] Method 1: Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`: "Method 1: Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`"
- [readme] It also supports FASTQ file analysis, filtering, conversion and merging of paired-end reads.: [evidence span withheld — anchor status: external_doc]
- [abstract] including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
