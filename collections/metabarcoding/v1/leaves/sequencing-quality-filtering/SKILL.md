---
name: sequencing-quality-filtering
description: Use when raw Illumina paired-end amplicon FASTQ reads must be merged,
  primer-trimmed, and expected-error-filtered before dereplication, chimera removal,
  or OTU/ASV clustering in an EasyAmplicon-style pipeline.
when_to_use_negative:
- Input is already a dereplicated FASTA or an OTU/ASV feature table — this skill operates
  on raw/merged paired-end FASTQ reads
- Primer sequences are unknown or of different lengths than the pipeline's 29 bp forward
  / 18 bp reverse assumption — the stripleft/stripright values must be re-derived
  first
- The analysis uses long-read PacBio/Nanopore amplicon data (EasyAmplicon 2 scope),
  which the merge-pairs step does not apply to
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: VSEARCH
  role: Merges paired-end reads (--fastq_mergepairs), relabels them by sample ID,
    and filters the concatenated FASTQ with --fastx_filter for primer stripping and
    expected-error filtering
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline that orchestrates the merge, primer-trim, and quality-filter steps
    producing temp/filtered.fa before OTU/ASV analysis
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:108c1020c6c56cc25e7cd67b5e2080d51d1a99ca4b9ce0a245f0b8588a689e15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# sequencing-quality-filtering

## Summary

Quality-filters merged paired-end amplicon reads with VSEARCH's fastx_filter: strips primer bases from both ends and discards reads exceeding an expected-error threshold, yielding clean FASTA amplicons (temp/filtered.fa) for downstream OTU/ASV analysis.

## When to use

Use when raw Illumina paired-end amplicon FASTQ reads must be merged, primer-trimmed, and expected-error-filtered before dereplication, chimera removal, or OTU/ASV clustering in an EasyAmplicon-style pipeline.

## When NOT to use

- Input is already a dereplicated FASTA or an OTU/ASV feature table — this skill operates on raw/merged paired-end FASTQ reads
- Primer sequences are unknown or of different lengths than the pipeline's 29 bp forward / 18 bp reverse assumption — the stripleft/stripright values must be re-derived first
- The analysis uses long-read PacBio/Nanopore amplicon data (EasyAmplicon 2 scope), which the merge-pairs step does not apply to

## Inputs

- Per-sample gzipped paired-end Illumina FASTQ reads (e.g. seq/Sample1_1.fq.gz, seq/Sample1_2.fq.gz)
- Concatenated merged FASTQ of all samples (temp/all.fq)
- Known forward/reverse primer lengths (e.g. 29 bp left, 18 bp right)

## Outputs

- Per-sample merged FASTQ files (temp/Sample1.merged.fq)
- Concatenated merged FASTQ (temp/all.fq)
- Primer-trimmed, quality-filtered FASTA of amplicons (temp/filtered.fa)

## How to apply

First merge paired-end reads per sample with vsearch --fastq_mergepairs (e.g. seq/Sample1_1.fq.gz with --reverse seq/Sample1_2.fq.gz), relabeling reads with the sample ID prefix via --relabel Sample1. to keep sample provenance, and write per-sample merged FASTQ files. Concatenate all per-sample merged files (cat temp/*.merged.fq > temp/all.fq) so filtering is applied uniformly. Then run vsearch --fastx_filter on temp/all.fq with --fastq_stripleft 29 and --fastq_stripright 18 to remove the 29 bp forward and 18 bp reverse primer bases, and --fastq_maxee_rate 0.01 so reads whose expected errors exceed 1% of read length are dropped; the article's alternative parameterization also shows length limits (--fastq_minlen 1200 --fastq_maxlen 1800) and absolute --fastq_maxee, plus cutadapt --discard-untrimmed for primer removal. Output the surviving amplicons as --fastaout temp/filtered.fa and validate by confirming the file exists and inspecting its first records with head.

## Related tools

- **VSEARCH** (Merges paired-end reads (--fastq_mergepairs), relabels them by sample ID, and filters the concatenated FASTQ with --fastx_filter for primer stripping and expected-error filtering) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline that orchestrates the merge, primer-trim, and quality-filter steps producing temp/filtered.fa before OTU/ASV analysis) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --fastx_filter temp/all.fq --fastq_stripleft 29 --fastq_stripright 18 --fastq_maxee_rate 0.01 --fastaout temp/filtered.fa
```

## Evaluation signals

- temp/filtered.fa exists and its first records, inspected with head, are correctly formatted FASTA with primer bases removed
- All retained read headers carry the sample-ID prefix (e.g. Sample1.) introduced by --relabel, preserving sample provenance
- Read lengths in temp/filtered.fa equal merged lengths minus 47 bp (29 bp left + 18 bp right primer trim)
- No sequence in the output would exceed the expected-error cutoff — reads with expected errors above 1% of read length (maxee_rate 0.01) have been discarded
- Downstream dereplication/OTU clustering consumes temp/filtered.fa without format errors

## Limitations

- The primer trim lengths (29 bp left, 18 bp right) are assay-specific; a different primer set requires recalculated stripleft/stripright values or sequence-aware trimming (e.g. cutadapt --discard-untrimmed)
- The article's methods variant shows alternative length constraints (--fastq_minlen 1200 --fastq_maxlen 1800) suited to long amplicons — default 16S settings must be adjusted for other targets
- Quality filtering alone does not remove chimeras; chimera removal (de novo via cluster_otus or reference-based uchime_ref) is a separate downstream step

## Evidence

- [methods] Method 1: Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`: "Method 1: Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`"
- [methods] Use `cutadapt` to specify both forward and reverse primer sequences (e.g. `-g "AGAGTTTGATCCTGGCTCAG...AAGTCSTAACAAGGTADCCSTA"`), `--discard-untrimmed`: "Primer removal**: Use `cutadapt` to specify both forward and reverse primer sequences (e.g. `-g "AGAGTTTGATCCTGGCTCAG...AAGTCSTAACAAGGTADCCSTA"`), `--discard-untrimmed`"
- [readme] It also supports FASTQ file analysis, filtering, conversion and merging of paired-end reads.: [evidence span withheld — anchor status: external_doc]
- [abstract] including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
