---
name: otu-taxonomic-classification-mapping
description: Use when 16S amplicon reads (e.g. V3-V4 FASTQ from MiSeq, already adapter-
  and quality-trimmed) must be clustered into OTUs and classified taxonomically against
  SILVA SSU ref release 132 to build an abundance table for downstream diversity or
  differential-abundance tests.
when_to_use_negative:
- Input is ITS/eukaryotic amplicon data — the paper classifies ITS consensus sequences
  with the 'Classify Sequences' tool using the Warcup V2 training data set (90% similarity,
  minimum overlap 75 bp), not the SILVAngs/BLASTn 16S route.
- Input is already a classified OTU/feature table — only the downstream relative-abundance
  and statistical testing steps apply.
- Raw unprocessed reads that still contain 3'-end adapters and low-quality tails —
  run Scythe before quality-based trimming first, since quality trimming could remove
  bases helpful for identifying adapter contaminants.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: SILVAngs
  role: Online automatic pipeline performing OTU clustering and taxonomic classification
    of 16S reads against the SILVA rDNA database
- name: SINA v1.2.10
  role: Aligns each read against SILVA SSU rRNA within the SILVAngs pipeline
- name: cd-hit-est v3.1.2
  role: Dereplicates and clusters reads into OTUs at identity criteria 1.00 and 0.98
- name: BLASTn
  role: Local search of OTUs against the non-redundant SILVA SSU ref release 132 to
    assign taxonomy
- name: BBduk
  role: Upstream quality trimming (Q30) and discarding of reads below 50 bp before
    classification
- name: Scythe
  role: Bayesian 3'-end adapter trimming of FASTQ reads upstream of OTU clustering
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Windowed adaptive quality trimming of FASTQ reads upstream of OTU clustering
  repo: https://github.com/najoshi/sickle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:8a46c7b130ce3a6249c740c4fe9df45c6e1b8eb49ae12ce56a4847b5d7f05401
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - BBduk
  - BLASTn
  - SILVAngs
  - SINA v1.2.10
  - Scythe
  - Sickle
  - XLSTAT 2017
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# otu-taxonomic-classification-mapping

## Summary

Cluster quality-trimmed 16S rRNA amplicon reads into OTUs and assign taxonomy by aligning each read against the SILVA SSU reference database, as done in the trout-farm metabarcoding study via the SILVAngs pipeline (SINA alignment, cd-hit-est clustering, local BLASTn classification).

## When to use

Use when 16S amplicon reads (e.g. V3-V4 FASTQ from MiSeq, already adapter- and quality-trimmed) must be clustered into OTUs and classified taxonomically against SILVA SSU ref release 132 to build an abundance table for downstream diversity or differential-abundance tests.

## When NOT to use

- Input is ITS/eukaryotic amplicon data — the paper classifies ITS consensus sequences with the 'Classify Sequences' tool using the Warcup V2 training data set (90% similarity, minimum overlap 75 bp), not the SILVAngs/BLASTn 16S route.
- Input is already a classified OTU/feature table — only the downstream relative-abundance and statistical testing steps apply.
- Raw unprocessed reads that still contain 3'-end adapters and low-quality tails — run Scythe before quality-based trimming first, since quality trimming could remove bases helpful for identifying adapter contaminants.

## Inputs

- Compressed FASTQ files of merged 16S V3-V4 amplicon reads (post adapter removal, Q30 trimming, and merging with BBmerge/BBduk)
- SILVA SSU ref database, non-redundant release no. 132 (BLASTn reference)
- SILVA rDNA database (SILVAngs default classification reference)

## Outputs

- OTU abundance table with taxonomic assignments at species/genus/family/phylum level
- Classification status per read/OTU (classified vs. unclassified/unassigned for weak BLAST hits)
- Dereplicated OTU representatives clustered at identity criteria 1.00 and 0.98

## How to apply

Upload compressed FASTQ files to the SILVAngs online automatic pipeline, which runs at default settings against the SILVA rDNA database. Each read is first aligned against SILVA SSU rRNA using SINA v1.2.10; identical reads are dereplicated and then clustered/assigned as OTUs using cd-hit-est v3.1.2 with identity criteria of 1.00 and 0.98 (ignoring overhangs). Taxonomy is assigned per OTU via a local BLASTn search against the non-redundant SILVA SSU ref dataset (release no. 132); reads with no or weak hits where (% sequence identity + % alignment coverage)/2 does not exceed 93 remain unclassified or unassigned. Upstream, reads must have adapters removed (Scythe v0.994, then Sickle), be trimmed for Q30 quality, and short reads (<50 bp after BBduk; sequences below 36 bp discarded) removed; SILVAngs additionally discards short reads and reads with more than 2% ambiguities or homopolymers. The result is an OTU table with taxonomic assignments used for downstream relative-abundance and t-test analyses (e.g. the I-source vs P-source species comparisons).

## Related tools

- **SILVAngs** (Online automatic pipeline performing OTU clustering and taxonomic classification of 16S reads against the SILVA rDNA database)
- **SINA v1.2.10** (Aligns each read against SILVA SSU rRNA within the SILVAngs pipeline)
- **cd-hit-est v3.1.2** (Dereplicates and clusters reads into OTUs at identity criteria 1.00 and 0.98)
- **BLASTn** (Local search of OTUs against the non-redundant SILVA SSU ref release 132 to assign taxonomy)
- **BBduk** (Upstream quality trimming (Q30) and discarding of reads below 50 bp before classification)
- **Scythe** (Bayesian 3'-end adapter trimming of FASTQ reads upstream of OTU clustering) — http://github.com/vsbuffalo/scythe
- **Sickle** (Windowed adaptive quality trimming of FASTQ reads upstream of OTU clustering) — https://github.com/najoshi/sickle

## Examples

```
sickle se -f input_file.fastq -t illumina -o trimmed_output_file.fastq -q 33 -l 40  # then upload trimmed FASTQ to SILVAngs (SINA -> cd-hit-est 1.00/0.98 -> BLASTn vs SILVA SSU ref 132)
```

## Evaluation signals

- Silva filter applied: short reads and reads with more than 2% ambiguities or homopolymers were discarded before clustering.
- OTUs must be clustered with cd-hit-est v3.1.2 at identity criteria 1.00 and 0.98, ignoring overhangs.
- Classification cutoff respected: reads/OTUs whose (% sequence identity + % alignment coverage)/2 does not exceed 93 must be reported as unclassified or unassigned.
- Reference database check: BLASTn hits must be against the non-redundant SILVA SSU ref dataset release no. 132.
- Resulting taxonomic profile should reproduce the paper's expectation that >99% of prokaryotic OTUs were bacteria (~75.57% Proteobacteria, ~14.4% Bacteroidetes).

## Limitations

- Samples with no or weak BLAST hits (score <= 93 on (% identity + % alignment coverage)/2) remain unclassified or unassigned, leaving some OTUs without taxonomy.
- The exact farm sampling locations cannot be disclosed at the farmers' request, limiting geographic reproducibility of the classification exercise.
- Scythe only checks for 3'-end contaminants and should run before quality-based trimming; it cannot handle 5'-end or internal contamination (TagDust is suggested for reads contaminated at any position).

## Evidence

- [discussion] SILVAngs pipeline: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] SINA alignment: "Each read was aligned against the SILVA SSU rRNA using the SILVA Incremental Aligner (SINA v1.2.10;51)"
- [discussion] OTU clustering: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2;52)"
- [discussion] BLASTn classification: "based on local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132)"
- [discussion] SILVA filter: "Short reads and/or with more than 2% ambiguities or homopolymers were discarded"
- [discussion] unclassified threshold: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned"
- [readme] Scythe rationale: "Scythe uses a Naive Bayesian approach to classify contaminant substrings in sequence reads."
