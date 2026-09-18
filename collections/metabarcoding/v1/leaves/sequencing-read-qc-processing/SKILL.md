---
name: sequencing-read-qc-processing
description: Use when raw Illumina MiSeq FASTQ reads (e.g. 16S V3-V4 or ITS2 amplicons)
  may carry 3'-end adapter contaminants and must be trimmed prior to OTU clustering
  or taxonomic classification.
when_to_use_negative:
- Contamination occurs in any read position (not just the 3'-end) — the README recommends
  TagDust for that case, since Scythe only checks 3'-end contaminants
- 5'-end adapter trimming is required — Scythe does not allow adapters overlapping
  the 5'-end
- Input is already a quality-trimmed FASTQ where low-quality bases were removed before
  adapter detection, undermining Bayesian classification
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Scythe
  role: Naive Bayesian adapter trimmer (v0.994 BETA) that classifies and removes 3'-end
    adapter contaminants from FASTQ reads, quality-aware
  repo: https://github.com/vsbuffalo/scythe
- name: Sickle
  role: Windowed adaptive quality-based trimming applied after adapter removal, discarding
    short reads
  repo: https://github.com/najoshi/sickle
- name: BBduk
  role: Subsequent low-quality (Q30) trimming and discarding of reads below 50 bp
    in Geneious Prime
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:cb947380b254f19d60e3b37e23123cb46abbd25615e8457c464c8e4793b8e20a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - BBduk
  - Scythe
  - Sickle
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# sequencing-read-qc-processing

## Summary

Bayesian 3'-end adapter trimming of metabarcoding FASTQ reads with Scythe v0.994, run before quality-based trimming (Sickle/BBduk), so that low-quality bases do not mask adapter contaminant evidence.

## When to use

Use when raw Illumina MiSeq FASTQ reads (e.g. 16S V3-V4 or ITS2 amplicons) may carry 3'-end adapter contaminants and must be trimmed prior to OTU clustering or taxonomic classification.

## When NOT to use

- Contamination occurs in any read position (not just the 3'-end) — the README recommends TagDust for that case, since Scythe only checks 3'-end contaminants
- 5'-end adapter trimming is required — Scythe does not allow adapters overlapping the 5'-end
- Input is already a quality-trimmed FASTQ where low-quality bases were removed before adapter detection, undermining Bayesian classification

## Inputs

- Raw paired-end FASTQ reads from MiSeq 3000 (16S V3-V4 and ITS2 amplicons)
- Adapter sequence FASTA file (Scythe repository test-data adapter file, with barcode placeholder characters usable as provided)

## Outputs

- trimmed_sequences.fastq containing input reads with 3'-end adapter contaminant bases removed
- matches.txt reporting contamination occurrence counts per adapter position

## How to apply

Compile Scythe with 'make all' (GCC/Clang; bundled kseq.h plus Zlib) and run it in Bayesian contamination-classification mode on the FASTQ reads with an adapter FASTA file, keeping the default prior contamination rate of 0.05 and the default Illumina quality scheme (pipeline > 1.3). Retain the default minimum match length (-n 5) so only contaminants matching at least 5 bases at the 3'-end are trimmed, and collect per-adapter contamination counts via the '-m matches.txt' file. Run Scythe BEFORE quality-based trimming (Sickle/BBduk), because quality-based trimming could remove bases that help identify 3'-end adapter contaminants. In this study's workflow, adapter removal with Scythe and Sickle was applied to 1,054,909 16S and 2,193,552 ITS2 reads, discarding sequences below 36 bp.

## Related tools

- **Scythe** (Naive Bayesian adapter trimmer (v0.994 BETA) that classifies and removes 3'-end adapter contaminants from FASTQ reads, quality-aware) — https://github.com/vsbuffalo/scythe
- **Sickle** (Windowed adaptive quality-based trimming applied after adapter removal, discarding short reads) — https://github.com/najoshi/sickle
- **BBduk** (Subsequent low-quality (Q30) trimming and discarding of reads below 50 bp in Geneious Prime)

## Examples

```
scythe -a adapter_file.fasta -o trimmed_sequences.fastq -m matches.txt sequences.fastq
```

## Evaluation signals

- trimmed_sequences.fastq is well-formed FASTQ containing the input reads with 3'-end adapter contaminant bases removed
- matches.txt reports contamination occurrence counts per adapter position consistent with the trimmed output
- Per-sample read counts reconcile with the sequencing output in supplementary table 1 (e.g. 1,054,909 16S reads and 2,193,552 ITS2 reads entering preprocessing)
- No reads shorter than the discarded minimum length (below 36 bp in this workflow) remain in the trimmed output
- Adapter-trimmed reads show no residual adapter sequence at read 3'-ends when screened downstream

## Limitations

- Scythe only checks for 3'-end contaminants and does not support 5'-end adapter trimming
- Default minimum match length (-n 5) on a very contaminated lane could lead to very liberal trimming if decreased
- The provided adapter sequence files contain non-FASTA barcode placeholder characters ([NNNNNN]/[MMMMMM]) that may need modification depending on library preparation
- Reads are not removed entirely (unlike TagDust); adapters are trimmed off while leaving uncontaminated sequence, which may leave very short fragments requiring downstream length filtering

## Evidence

- [discussion] Scythe (v0.994; available at http://github.com/vsbuffalo/scythe) and Sickle ... software were used to remove the adapter sequences: [evidence span withheld — anchor status: not_found]
- [methods] quality-based trimming could remove bases that are helpful in identifying (and removing) 3'-end adapter contaminants: "However, quality-based trimming could remove bases that are helpful in identifying (and removing) 3'-end adapter contaminants."
- [readme] Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads: "Scythe uses a Naive Bayesian approach to classify contaminant
substrings in sequence reads."
- [readme] By default, the prior contamination rate is 0.05: "By default, the prior contamination rate is 0.05."
- [discussion] were used to remove the adapter sequences, while sequences below 36 bp were discarded: "were used to remove the adapter sequences, while sequences below 36 bp were discarded"
