---
name: fastq-quality-aware-trimming
description: Use when raw paired-end Illumina FASTQ reads (e.g. MiSeq 16S/ITS metabarcoding
  data) still contain 3'-end adapter contamination and must be trimmed prior to merging,
  OTU clustering, or taxonomic classification.
when_to_use_negative:
- Contamination can occur at any read position, not just the 3'-end — the README recommends
  TagDust for that case
- 5'-end trimming is needed — Scythe only removes 3'-end contaminants and does not
  allow adapters to overlap the 5'-end
- Input is already merged/quality-trimmed — running Scythe after aggressive quality
  trimming can remove the low-quality bases it relies on for adapter detection
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Scythe v0.994
  role: Naive Bayesian 3'-end adapter trimmer applied to raw FASTQ reads with default
    prior contamination rate 0.05 and minimum match length 5
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Sliding-window, adaptive quality-based trimming of read ends after adapter
    removal
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:0c8f6dd312fae93d731ae66f09872af5ce2843a38a6921e322748137cc8553bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - Scythe
  - Scythe v0.994
  - Sickle
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# fastq-quality-aware-trimming

## Summary

Removes 3'-end adapter contaminants from raw metabarcoding FASTQ reads (16S V3-V4 and ITS2 libraries) using the Naive Bayesian trimmer Scythe v0.994, followed by windowed quality trimming with Sickle, before OTU clustering via SILVAngs.

## When to use

Use when raw paired-end Illumina FASTQ reads (e.g. MiSeq 16S/ITS metabarcoding data) still contain 3'-end adapter contamination and must be trimmed prior to merging, OTU clustering, or taxonomic classification.

## When NOT to use

- Contamination can occur at any read position, not just the 3'-end — the README recommends TagDust for that case
- 5'-end trimming is needed — Scythe only removes 3'-end contaminants and does not allow adapters to overlap the 5'-end
- Input is already merged/quality-trimmed — running Scythe after aggressive quality trimming can remove the low-quality bases it relies on for adapter detection

## Inputs

- Raw FASTQ reads from MiSeq 3000 sequencing (1,054,909 reads for 16S V3-V4 libraries; 2,193,552 for ITS2)
- Adapter sequence FASTA file (bundled with the Scythe repository test data, including barcode placeholder characters)

## Outputs

- Trimmed FASTQ (trimmed_sequences.fastq) with 3'-end adapter bases removed
- Per-adapter contamination occurrence counts by position (matches.txt via '-m', plus stderr report)
- Length- and quality-filtered reads ready for BBmerge/BBduk and downstream OTU clustering

## How to apply

Run Scythe BEFORE quality-based trimming, because quality trimmers can remove the low-quality 3'-end bases that actually help identify adapter contaminants. Build Scythe from source with 'make all' (GCC or Clang; requires bundled kseq.h and Zlib) and run it in Bayesian contamination-classification mode: 'scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq', keeping the default prior contamination rate of 0.05, the default Illumina quality scheme (pipeline > 1.3), and the default minimum match length (-n 5) so only 3'-end matches of at least 5 bases are trimmed. Scythe compares two likelihood models — matches given contamination (quality-weighted) versus given random sequence — and trims the class with maximum posterior probability. After adapter removal, apply Sickle's sliding-window adaptive quality trimming and discard short reads (the paper discarded sequences below 36 bp; its BBduk step applied Q30 trimming discarding reads below 50 bp). Collect the '-o' trimmed FASTQ and the per-adapter-position counts from the '-m' matches file and standard error.

## Related tools

- **Scythe v0.994** (Naive Bayesian 3'-end adapter trimmer applied to raw FASTQ reads with default prior contamination rate 0.05 and minimum match length 5) — http://github.com/vsbuffalo/scythe
- **Sickle** (Sliding-window, adaptive quality-based trimming of read ends after adapter removal) — https://github.com/najoshi/sickle

## Examples

```
scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq
```

## Evaluation signals

- trimmed_sequences.fastq is a well-formed FASTQ containing the input reads with 3'-end adapter bases removed
- matches.txt reports per-adapter contamination occurrence counts by position consistent with the trimmed output
- No sequences shorter than the length threshold (36 bp in this workflow) survive the trimming step
- Posterior contamination classification uses the default prior rate of 0.05 and Illumina quality scheme (pipeline > 1.3), i.e. no unintended '-p' or '-q' overrides
- Downstream BBmerge and SILVAngs clustering proceed on compressed FASTQ without untrimmed adapter overhangs

## Limitations

- Scythe only checks for 3'-end contaminants; it does not handle 5'-end adapters or adapters overlapping the 5'-end
- Adapter FASTA files bundled with Scythe contain non-FASTA barcode placeholder characters ([NNNNNN]/[MMMMMM]) that may need modification to match the library preparation
- Decreasing the minimum match length (-n, default 5) on a heavily contaminated lane can cause very liberal trimming of only a few bases
- Scythe works with paired-end data but each file must be run separately; it will not remove reads entirely, so mismatched pairs are not created

## Evidence

- [discussion] Scythe (v0.994; available at http://github.com/vsbuffalo/scythe) and Sickle: "Scythe (v0.994; available at http://github.com/vsbuffalo/scythe) and Sickle"
- [discussion] were used to remove the adapter sequences, while sequences below 36 bp were discarded: "were used to remove the adapter sequences, while sequences below 36 bp were discarded"
- [readme] Thus, it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline.: "Thus, it is recommended you run Scythe *before* quality-based trimming, as part of a read quality control pipeline."
- [readme] Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads.: "Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads."
