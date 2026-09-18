---
name: naive-bayes-contamination-classification
description: Use when raw Illumina FASTQ metabarcoding reads (16S V3-V4 or ITS2) contain
  3'-end adapter contaminants and quality-based trimming would erase the low-quality
  bases needed to identify those contaminants; run Scythe before quality trimming.
when_to_use_negative:
- 'Contamination can occur at any read position (not just the 3''-end): the README
  recommends TagDust for reads with contamination in any position, since Scythe only
  checks 3''-end contaminants and cannot trim 5''-end adapters'
- 'The goal is to discard contaminated reads entirely: Scythe trims off contaminating
  sequence and keeps the remaining read, unlike TagDust which removes whole reads'
- 'Input is already quality-trimmed: quality-based trimming may have removed the low-quality
  bases Scythe relies on to identify 3''-end adapters, so it should run before, not
  after, quality trimming'
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Scythe
  role: Naive Bayesian adapter trimmer (v0.994 BETA) that classifies and removes 3'-end
    adapter contaminants from FASTQ reads
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Windowed adaptive quality-based trimmer run alongside Scythe for adapter/quality
    removal; sequences below 36 bp were discarded
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:1128e7485fb11298cb4efcd5bab093d56a800203330562ae95a0672593390381
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - Scythe
  - Sickle
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# naive-bayes-contamination-classification

## Summary

Adapter trimming of metabarcoding FASTQ reads using Scythe v0.994, a Naive Bayesian classifier that removes 3'-end adapter contaminants while tolerating mismatches caused by poor base quality. In the trout-farm metabarcoding study it cleaned 1,054,909 16S reads and 2,193,552 ITS2 reads before OTU clustering.

## When to use

Use when raw Illumina FASTQ metabarcoding reads (16S V3-V4 or ITS2) contain 3'-end adapter contaminants and quality-based trimming would erase the low-quality bases needed to identify those contaminants; run Scythe before quality trimming.

## When NOT to use

- Contamination can occur at any read position (not just the 3'-end): the README recommends TagDust for reads with contamination in any position, since Scythe only checks 3'-end contaminants and cannot trim 5'-end adapters
- The goal is to discard contaminated reads entirely: Scythe trims off contaminating sequence and keeps the remaining read, unlike TagDust which removes whole reads
- Input is already quality-trimmed: quality-based trimming may have removed the low-quality bases Scythe relies on to identify 3'-end adapters, so it should run before, not after, quality trimming

## Inputs

- Raw Illumina FASTQ reads (paired-end MiSeq 3000 output: 16S V3-V4 and ITS2 amplicons)
- Adapter sequence FASTA file (e.g. the adapter files supplied with the Scythe repository test data, containing barcode placeholder characters)

## Outputs

- Trimmed FASTQ file (3'-end adapter contaminant bases removed), e.g. trimmed_sequences.fastq
- matches.txt reporting contamination occurrence counts per adapter position

## How to apply

Load the FASTQ reads together with the adapter sequence FASTA file from the Scythe repository test data, keeping the barcode placeholder characters ([NNNNNN]/[MMMMMM]) as provided. Build the executable with 'make all' (GCC or Clang; the tool relies on bundled kseq.h and Zlib). Run Scythe in Bayesian contamination-classification mode with 'scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq', keeping the default prior contamination rate of 0.05 and the default Illumina quality scheme (pipeline > 1.3). Retain the default minimum match length (-n, default 5) so only 3'-end contaminants matching at least 5 bases are trimmed; because quality-based trimming (e.g. with Sickle/BBduk, Q30) could remove bases helpful for identifying adapters, Scythe should run first in the QC pipeline. Collect the trimmed reads from the '-o' output and per-adapter contamination counts by position from the '-m' matches file and standard error.

## Related tools

- **Scythe** (Naive Bayesian adapter trimmer (v0.994 BETA) that classifies and removes 3'-end adapter contaminants from FASTQ reads) — http://github.com/vsbuffalo/scythe
- **Sickle** (Windowed adaptive quality-based trimmer run alongside Scythe for adapter/quality removal; sequences below 36 bp were discarded) — https://github.com/najoshi/sickle

## Examples

```
scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq
```

## Evaluation signals

- trimmed_sequences.fastq is well-formed FASTQ containing the input reads with 3'-end adapter contaminant bases removed
- matches.txt reports per-adapter contamination occurrence counts by position consistent with the number of trimmed reads
- No reads were removed entirely — Scythe trims contaminating sequence while leaving the uncontaminated portion of each read
- Default parameters hold: prior contamination rate 0.05, Illumina quality scheme (pipeline > 1.3), minimum match length 5 bases
- Output read counts remain compatible with the study's input volumes (1,054,909 16S reads; 2,193,552 ITS2 reads) after trimming with only short reads (<36 bp) discarded downstream

## Limitations

- Scythe only checks for 3'-end contaminants; it does not trim 5'-end adapters and does not allow adapters to overlap the 5'-end
- The bundled adapter FASTA files contain non-FASTA barcode placeholder characters ([NNNNNN]/[MMMMMM]) that must be modified/usable as provided for the specific library preparation
- In very contaminated lanes, decreasing the minimum match length (-n) below the default 5 could lead to very liberal trimming of only a few bases
- For quality-based trimming downstream, beware that it could remove bases that are helpful in identifying (and removing) 3'-end adapter contaminants, so run Scythe first

## Evidence

- [discussion] Scythe (v0.994; available at http://github.com/vsbuffalo/scythe) and Sickle: "Scythe (v0.994; available at http://github.com/vsbuffalo/scythe) and Sickle"
- [discussion] were used to remove the adapter sequences, while sequences below 36 bp were discarded: "were used to remove the adapter sequences, while sequences below 36 bp were discarded"
- [methods] Scythe just removes 3'-end contaminants where there could be multiple base mismatches due to poor base quality.: "Scythe just removes 3'-end contaminants where there could be multiple base mismatches due to poor base quality."
- [readme] Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads.: "Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads."
- [readme] quality-based trimming could remove bases that are helpful in identifying (and removing) 3'-end adapter contaminants.: "quality-based trimming could remove bases that are helpful in identifying (and removing) 3'-end adapter contaminants."
