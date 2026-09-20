---
name: paired-end-read-merging
description: Use when starting from per-sample paired-end gzipped FASTQ amplicon reads
  (e.g. Sample1_1.fq.gz/Sample1_2.fq.gz) and you need merged, primer-trimmed, quality-filtered
  amplicons before dereplication and OTU/ASV inference.
when_to_use_negative:
- Input is already a merged, dereplicated FASTA/feature table — merging and primer
  stripping have already been performed
- Long-read PacBio/Nanopore amplicon data (EasyAmplicon 2 handles these with a different
  sub-pipeline)
- Reads whose primer/amplicon lengths differ from the 29 bp/18 bp trim used here —
  trim lengths must be adjusted to the actual primer set
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: VSEARCH
  role: Performs paired-end merging (--fastq_mergepairs), primer stripping and expected-error
    filtering (--fastx_filter, --fastq_maxee_rate 0.01) to yield temp/filtered.fa
  repo: https://github.com/torognes/vsearch
- name: EasyAmplicon
  role: Pipeline in which the merge/primer-trim/filter step (sections 6.2.2-6.2.3)
    is scripted and its outputs (temp/all.fq, temp/filtered.fa) are staged for downstream
    OTU/ASV analysis
  repo: https://github.com/YongxinLiu/EasyAmplicon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imo2.42
    title: 'Unveiling microbial communities with EasyAmplicon: A user‐centric guide
      to perform amplicon sequencing data analysis'
  source_content_hash: sha256:fbd1a981c444db81223a96f0644698967d02535345132266e71e544e0341cc33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - EasyAmplicon
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# paired-end-read-merging

## Summary

Merges Illumina paired-end amplicon reads (e.g. seq/Sample1_1.fq.gz + seq/Sample1_2.fq.gz) into single sequences with VSEARCH's --fastq_mergepairs in EasyAmplicon, then strips primers (29 bp left / 18 bp right) and quality-filters by expected error to produce clean amplicon FASTA (temp/filtered.fa).

## When to use

Use when starting from per-sample paired-end gzipped FASTQ amplicon reads (e.g. Sample1_1.fq.gz/Sample1_2.fq.gz) and you need merged, primer-trimmed, quality-filtered amplicons before dereplication and OTU/ASV inference.

## When NOT to use

- Input is already a merged, dereplicated FASTA/feature table — merging and primer stripping have already been performed
- Long-read PacBio/Nanopore amplicon data (EasyAmplicon 2 handles these with a different sub-pipeline)
- Reads whose primer/amplicon lengths differ from the 29 bp/18 bp trim used here — trim lengths must be adjusted to the actual primer set

## Inputs

- Per-sample paired-end gzipped FASTQ amplicon reads (e.g. seq/Sample1_1.fq.gz, seq/Sample1_2.fq.gz)
- Known forward/reverse primer lengths (29 bp left, 18 bp right for the tutorial amplicon)

## Outputs

- Per-sample merged FASTQ files (temp/Sample1.merged.fq) with --relabel sample-ID prefixes
- Pooled merged reads file (temp/all.fq)
- Primer-trimmed, quality-filtered amplicon FASTA (temp/filtered.fa)

## How to apply

For each sample, run vsearch --fastq_mergepairs on the forward read file with --reverse pointing to the mate file (e.g. --fastq_mergepairs seq/Sample1_1.fq.gz --reverse seq/Sample1_2.fq.gz --relabel Sample1. --fastqout temp/Sample1.merged.fq), so merged read headers are relabeled with the sample ID prefix for later sample attribution. Concatenate all per-sample merged FASTQ files into one pool (cat temp/*.merged.fq > temp/all.fq). Then run vsearch --fastx_filter on temp/all.fq with --fastq_stripleft 29 and --fastq_stripright 18 to remove the 29 bp forward and 18 bp reverse primer bases, --fastq_maxee_rate 0.01 to discard reads whose expected errors exceed 1% of read length, and --fastaout temp/filtered.fa. Note the paper also documents an alternative length/expected-error filtering style (vsearch --fastx_filter with --fastq_minlen/--fastq_maxlen/--fastq_maxee), so primer lengths and length bounds should match the actual primer set and amplicon region used.

## Related tools

- **VSEARCH** (Performs paired-end merging (--fastq_mergepairs), primer stripping and expected-error filtering (--fastx_filter, --fastq_maxee_rate 0.01) to yield temp/filtered.fa) — https://github.com/torognes/vsearch
- **EasyAmplicon** (Pipeline in which the merge/primer-trim/filter step (sections 6.2.2-6.2.3) is scripted and its outputs (temp/all.fq, temp/filtered.fa) are staged for downstream OTU/ASV analysis) — https://github.com/YongxinLiu/EasyAmplicon

## Examples

```
vsearch --fastq_mergepairs seq/Sample1_1.fq.gz --reverse seq/Sample1_2.fq.gz --relabel Sample1. --fastqout temp/Sample1.merged.fq; cat temp/*.merged.fq > temp/all.fq; vsearch --fastx_filter temp/all.fq --fastq_stripleft 29 --fastq_stripright 18 --fastq_maxee_rate 0.01 --fastaout temp/filtered.fa
```

## Evaluation signals

- temp/filtered.fa exists and its first records (inspect with 'head') are correctly formatted FASTA
- Every sequence in temp/filtered.fa is 47 bp shorter than its pre-trim merged read length (29 + 18 primer bases removed)
- Merged read headers carry the sample ID prefix from --relabel (e.g. 'Sample1.'), enabling correct downstream per-sample OTU tables
- Reads rejected by --fastq_maxee_rate 0.01 (expected error > 1% of read length) are absent from temp/filtered.fa
- Each temp/Sample1.merged.fq produced by --fastq_mergepairs contains only reads successfully paired between forward and reverse mates

## Limitations

- Primer trim lengths (29 bp left, 18 bp right) and maxee threshold (0.01) are specific to the EasyAmplicon tutorial amplicon; other primer sets/regions require different values
- Merging depends on sufficient overlap between mates; unpaired reads are dropped and contribute no downstream observations
- The article notes dependencies come with version requirements (e.g. VSEARCH v2.7.1+52 or later), so older versions may not support the same option set

## Evidence

- [readme] VSEARCH README on FASTQ support: [evidence span withheld — anchor status: external_doc]
- [methods] Article methods on vsearch fastx_filter parameters: "Use `vsearch --fastx_filter` with `--fastq_minlen 1200 --fastq_maxlen 1800`, along with `--fastq_maxee` (maximum expected error) and `--fastq_qmax`"
- [abstract] Article abstract on preprocessing stage: "including data preprocessing (quality filtering, chimera removal), amplicon sequence variant analysis, diversity analysis, and data visualization"
- [intro] Article intro on VSEARCH version: "VSEARCH v2.7.1+52 or later: VSEARCH is an open‐source tool for performing tasks such as sequence clustering, chimera detection, and dereplication."
