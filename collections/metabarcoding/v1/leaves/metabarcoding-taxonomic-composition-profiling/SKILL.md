---
name: metabarcoding-taxonomic-composition-profiling
description: Use when raw paired-end amplicon FASTQ reads (16S V3-V4 or ITS2) from
  environmental water samples must be converted into OTU-level taxonomic composition
  tables with kingdom- or functional-group-level percentages.
when_to_use_negative:
- Input is already a clustered, taxonomy-annotated OTU/feature table rather than raw
  amplicon FASTQ reads
- The goal is strain-level or WGS metagenomic profiling (e.g., characterizing feed
  microbial composition via whole-genome shotgun metagenomics), which the article
  explicitly proposes as a separate follow-up
- 'Reads contain contaminants in internal or 5'' positions: Scythe only removes 3''-end
  contaminants, and a tool like TagDust is recommended instead'
tools:
- name: Scythe v0.994
  role: Bayesian 3'-end adapter trimmer run before quality trimming
  repo: http://github.com/vsbuffalo/scythe
- name: Sickle
  role: Sliding-window, quality-based trimming of 3' and 5' read ends
  repo: https://github.com/najoshi/sickle
- name: BBmerge
  role: Merging paired-end FASTQ files at a normal merge rate
- name: BBduk (in Geneious Prime)
  role: Trimming to Q30 and discarding reads below 50 bp
- name: SILVAngs
  role: Online automatic pipeline for 16S OTU analysis against the SILVA rDNA database
- name: SINA v1.2.10
  role: Aligning each read against the SILVA SSU rRNA database
- name: cd-hit-est v3.1.2
  role: Dereplication and OTU clustering at identity criteria 1.00 and 0.98
- name: BLASTn
  role: Local similarity search against the non-redundant SILVA SSU ref dataset (release
    no. 132) for taxonomy assignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:3b7d7ffed5cced04a5a0c3e8fd41e336799212d18e368f6e801a6480bca1df5a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/vsbuffalo/scythe
  tools:
  - BBduk (in Geneious Prime)
  - BBmerge
  - BLASTn
  - SILVAngs
  - SINA v1.2.10
  - Scythe v0.994
  - Sickle
  - cd-hit-est v3.1.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# metabarcoding-taxonomic-composition-profiling

## Summary

Profile taxonomic composition of 16S rRNA and ITS metabarcoding communities by QC-trimming paired-end reads, clustering reads into OTUs, classifying against SILVA SSU and ITS reference data, and summarizing community composition (e.g., kingdom-level shares, functional groups) as counts and percentages.

## When to use

Use when raw paired-end amplicon FASTQ reads (16S V3-V4 or ITS2) from environmental water samples must be converted into OTU-level taxonomic composition tables with kingdom- or functional-group-level percentages.

## When NOT to use

- Input is already a clustered, taxonomy-annotated OTU/feature table rather than raw amplicon FASTQ reads
- The goal is strain-level or WGS metagenomic profiling (e.g., characterizing feed microbial composition via whole-genome shotgun metagenomics), which the article explicitly proposes as a separate follow-up
- Reads contain contaminants in internal or 5' positions: Scythe only removes 3'-end contaminants, and a tool like TagDust is recommended instead

## Inputs

- Paired-end Illumina FASTQ files of 16S V3-V4 and ITS2 amplicons from inflow- and pond-water samples
- Illumina adapter sequence file (FASTA) for Scythe
- Reference databases: SILVA SSU rDNA / SILVA SSU ref release 132 and Warcup V2 ITS training data set

## Outputs

- OTU table with taxonomy assignments for prokaryotic (16S) and eukaryotic (ITS) communities
- Taxonomic composition summary: total and known-species OTU counts (e.g., 233 eukaryotic OTUs, 118 known species) and percentage breakdowns per kingdom and functional group
- Count of unclassified/unassigned OTUs with weak BLAST hits

## How to apply

Start with adapter removal using Scythe v0.994 before quality trimming (quality-based trimming can remove bases that identify 3'-end adapters), then trim with Sickle using a sliding window and discard reads below 36 bp; merge pairs with BBmerge and trim to Q30 with BBduk while discarding reads below 50 bp. For 16S, process compressed fastq through the SILVAngs online pipeline against the SILVA SSU rRNA database, aligning with SINA v1.2.10, dereplicating and clustering OTUs with cd-hit-est v3.1.2 at identity criteria of 1.00 and 0.98, then BLASTn against the non-redundant SILVA SSU ref dataset release 132; hits where (% sequence identity + % alignment coverage)/2 does not exceed 93 are left unclassified. For ITS2, generate consensus sequences and classify with the 'Classify Sequences' tool against the Warcup V2 training data set, matching at 90% similarity with a minimum 75 bp overlap. Summarize outputs as OTU totals (e.g., 233 eukaryotic OTUs, 118 known species), kingdom percentages (Fungi ~55%, Plantae ~33%, Animalia ~12%), and fungal functional-group proportions (plant pathogens ~45%, mushrooms/yeasts ~28%), reporting unassigned OTUs as a composition caveat.

## Related tools

- **Scythe v0.994** (Bayesian 3'-end adapter trimmer run before quality trimming) — http://github.com/vsbuffalo/scythe
- **Sickle** (Sliding-window, quality-based trimming of 3' and 5' read ends) — https://github.com/najoshi/sickle
- **BBmerge** (Merging paired-end FASTQ files at a normal merge rate)
- **BBduk (in Geneious Prime)** (Trimming to Q30 and discarding reads below 50 bp)
- **SILVAngs** (Online automatic pipeline for 16S OTU analysis against the SILVA rDNA database)
- **SINA v1.2.10** (Aligning each read against the SILVA SSU rRNA database)
- **cd-hit-est v3.1.2** (Dereplication and OTU clustering at identity criteria 1.00 and 0.98)
- **BLASTn** (Local similarity search against the non-redundant SILVA SSU ref dataset (release no. 132) for taxonomy assignment)

## Examples

```
scythe -a adapter_file.fasta -o trimmed_sequences.fastq sequences.fastq && sickle se -f trimmed_sequences.fastq -t illumina -o quality_trimmed.fastq
```

## Evaluation signals

- QC statistics confirm adapter and quality trimming occurred: reads shorter than 36 bp (after Scythe/Sickle) and 50 bp (after BBduk) are absent from the merged reads
- OTU clustering respects the stated identity criteria of 1.00 and 0.98 in cd-hit-est v3.1.2
- Classified read counts are consistent: sequences whose (% sequence identity + % alignment coverage)/2 score does not exceed 93 must be reported as unclassified or unassigned
- The composition table reproduces the reported values (233 total eukaryotic OTUs, 118 known species) and kingdom percentages within the rounded '~' ranges (Fungi ~55%, Plantae ~33%, Animalia ~12%)
- Prokaryotic composition is dominated by bacteria (>99% of prokaryotic OTUs, ~75.57% Proteobacteria, ~14.4% Bacteroidetes)

## Limitations

- Some OTUs remained unclassified or unassigned due to no or weak BLAST hits (score threshold 93), so composition percentages understate total diversity
- Exact farm locations were withheld at farmers' request, limiting geographic reproducibility
- Scythe only checks for 3'-end contaminants and does not handle 5'-end or internal adapter contamination
- ITS classification at 90% similarity with a 75 bp minimum overlap is approximate; solexa quality encoding in trimming tools is a non-linear approximation

## Evidence

- [abstract] Of 233 eukaryotic OTUs, 118 were known species; ~45% were plant pathogens and ~28% were mushroom/yeasts: "118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts"
- [discussion] SILVA filter discards short reads and >2% ambiguities or homopolymers: "Short reads and/or with more than 2% ambiguities or homopolymers were discarded"
- [discussion] Weak BLAST hits below score 93 remain unclassified: "did not exceed the value of 93 remained unclassified or unassigned"
- [discussion] ITS classification thresholds: "find matches with 90% similarity) and a minimum overlap of 75 bp"
- [readme] Scythe README: run adapter trimming before quality trimming: "Thus, it is recommended you run Scythe *before*
quality-based trimming, as part of a read quality control pipeline."
- [readme] Sickle README: sliding-window quality trimming method: "It takes the quality
values and slides a window across them whose length is 0.1 times the
length of the read."
