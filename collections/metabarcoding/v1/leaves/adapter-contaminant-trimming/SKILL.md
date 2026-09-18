---
name: adapter-contaminant-trimming
description: Use when raw FASTQ metabarcoding reads (e.g. 16S or ITS2 amplicons) may
  carry 3'-end adapter contamination and must be cleaned before quality trimming,
  merging, and OTU clustering or taxonomic classification.
when_to_use_negative:
- Contamination can occur at any read position, not just the 3'-end — use a tool like
  TagDust, which removes contaminated reads entirely
- The goal is 5'-end trimming — Scythe does not allow adapters to overlap the 5'-end
  and should not be used for 5'-trimming
- Reads have already been quality-trimmed — quality-based trimming could remove bases
  helpful for identifying 3'-end adapter contaminants, so Scythe should precede it,
  not follow it
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: Scythe v0.994
  role: Naive Bayesian adapter trimmer that classifies and removes 3'-end adapter
    contaminants from sequencing reads, quality-aware via a posterior comparison of
    contaminated vs. non-contaminated likelihood models
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Used after Scythe in the pipeline for windowed, quality-based trimming of
    the FASTQ reads
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:ff2747db0a52a5001c5c336f88f4f4ea5da3f91b973319d5f9e246908f3ae866
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - Scythe
  - Scythe v0.994
  - Sickle
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# adapter-contaminant-trimming

## Summary

Removes 3'-end adapter contaminants from raw metabarcoding FASTQ reads using Scythe, a Naive Bayesian adapter trimmer (v0.994 BETA), before quality trimming and OTU clustering. Scythe is quality-aware, so it can detect adapters even where poor-quality bases cause mismatches.

## When to use

Use when raw FASTQ metabarcoding reads (e.g. 16S or ITS2 amplicons) may carry 3'-end adapter contamination and must be cleaned before quality trimming, merging, and OTU clustering or taxonomic classification.

## When NOT to use

- Contamination can occur at any read position, not just the 3'-end — use a tool like TagDust, which removes contaminated reads entirely
- The goal is 5'-end trimming — Scythe does not allow adapters to overlap the 5'-end and should not be used for 5'-trimming
- Reads have already been quality-trimmed — quality-based trimming could remove bases helpful for identifying 3'-end adapter contaminants, so Scythe should precede it, not follow it

## Inputs

- Raw paired-end FASTQ reads (e.g. 1,054,909 reads per 16S library; 2,193,552 reads for ITS2) from MiSeq sequencing
- Adapter sequence FASTA file (e.g. the TruSeq/Solexa-Illumina adapter files supplied with the Scythe repository test data)

## Outputs

- Trimmed FASTQ file (trimmed_sequences.fastq) containing input reads with 3'-end adapter contaminant bases removed
- Matches file (matches.txt) reporting contamination occurrence counts per adapter position

## How to apply

Compile Scythe with 'make all' (GCC or Clang; requires bundled kseq.h and Zlib) and run it in Bayesian contamination-classification mode: 'scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq'. Keep the default prior contamination rate of 0.05, the default Illumina quality scheme (pipeline > 1.3), and the default minimum match length (-n 5), so only contaminants matching at least 5 bases at the 3'-end are trimmed. Run Scythe BEFORE quality-based trimming (e.g. Sickle or BBduk) because quality trimming could remove bases that help identify 3'-end adapters. Collect the trimmed FASTQ from the '-o' output and per-adapter contamination counts by position from the '-m' matches file and standard error. Validate that the trimmed output is well-formed FASTQ with 3'-end adapter bases removed and that matches.txt counts are consistent with the trimming performed.

## Related tools

- **Scythe v0.994** (Naive Bayesian adapter trimmer that classifies and removes 3'-end adapter contaminants from sequencing reads, quality-aware via a posterior comparison of contaminated vs. non-contaminated likelihood models) — http://github.com/vsbuffalo/scythe
- **Sickle** (Used after Scythe in the pipeline for windowed, quality-based trimming of the FASTQ reads) — https://github.com/najoshi/sickle

## Examples

```
scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq
```

## Evaluation signals

- trimmed_sequences.fastq parses as well-formed FASTQ and contains the input reads with 3'-end adapter contaminant bases removed
- matches.txt reports contamination occurrence counts per adapter position consistent with the trimmed output
- Default settings held: prior contamination rate 0.05, Illumina quality scheme (pipeline > 1.3), minimum match length -n 5
- Downstream read counts (e.g. 1,054,909 for 16S; 2,193,552 for ITS2 inputs) remain traceable through the trimming step, with only adapter bases (not whole reads) removed
- Scythe ran before quality trimming (Sickle/BBduk), matching the recommended read QC pipeline order

## Limitations

- Scythe only checks for 3'-end contaminants and does not allow adapters to overlap the 5'-end
- Scythe has no FDR control unlike TagDust, which however removes contaminated reads entirely rather than trimming them
- Provided adapter files contain non-FASTA barcode placeholder characters ([NNNNNN]/[MMMMMM]) that may need modification depending on library preparation
- Decreasing the minimum match length (-n) on a very contaminated lane could lead to very liberal trimming of only a few bases

## Evidence

- [discussion] Scythe (v0.994; available at http://github.com/vsbuffalo/scythe) and Sickle ... software were used to remove the adapter sequences: [evidence span withheld — anchor status: not_found]
- [readme] Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads.: "Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads. It considers quality information, which can make it robust in picking out 3'-end adapters"
- [readme] it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline.: "Thus, it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline."
- [methods] Scythe just removes 3'-end contaminants where there could be multiple base mismatches due to poor base quality.: "Scythe just removes 3'-end contaminants where there could be multiple base mismatches due to poor base quality."
