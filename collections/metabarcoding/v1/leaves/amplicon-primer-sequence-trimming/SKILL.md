---
name: amplicon-primer-sequence-trimming
description: Use when merged paired-end 16S amplicon FASTQ reads still carry primer
  bases (e.g. 29 bp forward and 18 bp reverse in EasyAmplicon) and must be trimmed
  and quality-filtered before dereplication and OTU/ASV inference.
when_to_use_negative:
- Input reads have already been primer-trimmed and quality-filtered (e.g. output of
  a prior --fastx_filter step); re-trimming would remove biological bases.
- Primer sequences or trim lengths differ from the EasyAmplicon defaults (29 bp left,
  18 bp right) — trim coordinates must be re-derived from the actual primers, e.g.
  via cutadapt.
- Data are long-read PacBio/Nanopore amplicons from EasyAmplicon 2 rather than Illumina/BGI
  paired-end short reads, for which this merge-and-trim step is designed.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: VSEARCH
  role: Merges paired-end reads (--fastq_mergepairs), strips primer bases (--fastq_stripleft/--fastq_stripright),
    and quality-filters by expected error rate (--fastq_maxee_rate) via --fastx_filter
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline embedding the merge/trim/filter preprocessing step; parameterizes
    primer trim lengths and file layout (temp/*.merged.fq, temp/filtered.fa)
  repo: https://github.com/YongxinLiu/EasyAmplicon
- name: cutadapt
  role: 'Alternative primer-removal tool: specify forward and reverse primer sequences
    with -g and --discard-untrimmed to drop untrimmed reads'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:df4df8587990e35bce6fec0d90e8a933d0491b811edb86bd251129425137ce3d
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
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# amplicon-primer-sequence-trimming

## Summary

Strips primer sequences from merged paired-end amplicon reads and applies expected-error quality filtering using VSEARCH, as implemented in EasyAmplicon's preprocessing module (sections 6.2.2–6.2.3), producing clean amplicons in FASTA for downstream OTU/ASV analysis.

## When to use

Use when merged paired-end 16S amplicon FASTQ reads still carry primer bases (e.g. 29 bp forward and 18 bp reverse in EasyAmplicon) and must be trimmed and quality-filtered before dereplication and OTU/ASV inference.

## When NOT to use

- Input reads have already been primer-trimmed and quality-filtered (e.g. output of a prior --fastx_filter step); re-trimming would remove biological bases.
- Primer sequences or trim lengths differ from the EasyAmplicon defaults (29 bp left, 18 bp right) — trim coordinates must be re-derived from the actual primers, e.g. via cutadapt.
- Data are long-read PacBio/Nanopore amplicons from EasyAmplicon 2 rather than Illumina/BGI paired-end short reads, for which this merge-and-trim step is designed.

## Inputs

- Per-sample paired-end FASTQ files (e.g. seq/Sample1_1.fq.gz and seq/Sample1_2.fq.gz)
- Concatenated merged FASTQ file (temp/all.fq)
- Known forward/reverse primer lengths (29 bp and 18 bp in EasyAmplicon sections 6.2.2–6.2.3)

## Outputs

- Per-sample merged FASTQ files (temp/Sample1.merged.fq)
- Quality-filtered, primer-trimmed FASTA of amplicons (temp/filtered.fa)

## How to apply

First merge paired-end reads per sample with `vsearch --fastq_mergepairs` (e.g. seq/Sample1_1.fq.gz with --reverse seq/Sample1_2.fq.gz), relabeling reads with the sample ID via --relabel Sample1. and writing temp/Sample1.merged.fq. Concatenate all per-sample merged files with 'cat temp/*.merged.fq > temp/all.fq'. Then run `vsearch --fastx_filter` on temp/all.fq with --fastq_stripleft 29 and --fastq_stripright 18 to remove the 29 bp forward and 18 bp reverse primer bases, --fastq_maxee_rate 0.01 to drop reads whose expected errors exceed 1% of read length, and --fastaout temp/filtered.fa. Primer positions must match the actual primers used (cutadapt with explicit primer sequences is an alternative described in the article's methods); the maxee-rate threshold trades read retention against error burden for OTU/ASV inference.

## Related tools

- **VSEARCH** (Merges paired-end reads (--fastq_mergepairs), strips primer bases (--fastq_stripleft/--fastq_stripright), and quality-filters by expected error rate (--fastq_maxee_rate) via --fastx_filter) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline embedding the merge/trim/filter preprocessing step; parameterizes primer trim lengths and file layout (temp/*.merged.fq, temp/filtered.fa)) — https://github.com/YongxinLiu/EasyAmplicon
- **cutadapt** (Alternative primer-removal tool: specify forward and reverse primer sequences with -g and --discard-untrimmed to drop untrimmed reads)

## Examples

```
vsearch --fastx_filter temp/all.fq --fastq_stripleft 29 --fastq_stripright 18 --fastq_maxee_rate 0.01 --fastaout temp/filtered.fa
```

## Evaluation signals

- temp/filtered.fa exists and 'head' of the file shows correctly formatted FASTA records
- Trimmed read lengths equal merged lengths minus 29 bp (left) and 18 bp (right), consistent with the primer bases removed
- No retained read has an expected error rate exceeding 1% of read length (--fastq_maxee_rate 0.01)
- Merged per-sample FASTQs carry sample-ID-prefixed read labels (via --relabel Sample1.) so sample provenance survives the concat step
- Read counts drop from temp/all.fq to temp/filtered.fa by roughly the fraction of low-quality/over-long-expected-error reads, not to near zero

## Limitations

- Trim coordinates (29 bp left / 18 bp right) are hard-coded to EasyAmplicon's example primers; other primer sets require re-deriving offsets or using cutadapt with explicit primer sequences.
- Using --fastq_maxee_rate 0.01 discards reads whose expected errors exceed 1% of read length, so overly stringent thresholds can lose legitimate low-quality-length reads; per the article, alternative --fastx_filter setups use --fastq_minlen 1200 --fastq_maxlen 1800 and absolute --fastq_maxee instead.
- The article notes this generic preprocessing step is only one filter stage; subsequent chimera removal, non-bacteria filtering, and rarefaction are handled by separate steps/tools.

## Evidence

- [methods] Merged reads (temp/all.fq) are filtered with vsearch --fastx_filter, stripping 29 bp from the left and 18 bp from the right (primers) with a max expected error rate of 0.01: [evidence span withheld — anchor status: not_found]
- [methods] Method 1: Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`: "Method 1: Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`"
- [methods] Primer removal: Use `cutadapt` to specify both forward and reverse primer sequences (e.g. `-g "AGAGTTTGATCCTGGCTCAG...AAGTCSTAACAAGGTADCCSTA"`), `--discard-untrimmed`: "**Primer removal**: Use `cutadapt` to specify both forward and reverse primer sequences (e.g. `-g "AGAGTTTGATCCTGGCTCAG...AAGTCSTAACAAGGTADCCSTA"`), `--discard-untrimmed`"
- [abstract] including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
- [readme] It also supports FASTQ file analysis, filtering, conversion and merging of paired-end reads.: [evidence span withheld — anchor status: external_doc]
