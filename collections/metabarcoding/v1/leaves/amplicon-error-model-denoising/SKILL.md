---
name: amplicon-error-model-denoising
description: Use when raw paired-end Illumina FASTQ amplicon reads (e.g. mitochondrial
  16S rRNA or COI metabarcoding) must be denoised into per-marker MOTU count tables
  with error modelling and chimera removal.
when_to_use_negative:
- Input is already a denoised or feature/MOTU count table rather than raw paired-end
  FASTQ reads
- Reads are single-end or the amplicon design cannot produce at least a 5 bp overlap
  between forward and reverse reads
- A reference-based (closed-labeling) OTU clustering approach is required instead
  of de novo sequence-inference denoising
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: R package used for filterAndTrim, learnErrors, dada denoising, mergePairs
    (min 5 bp overlap) and removeBimeraDenovo to infer MOTUs from raw NextSeq reads
  repo: tobiasgf/Bioinformatic-tools
- name: R
  role: Scripting environment in which the DADA2 workflow is run to produce the MOTU
    count table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:531933acba9c7823fd327bd23b09df090445fa8e1645f0bb6bebaaee8e65bc8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - R
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# amplicon-error-model-denoising

## Summary

Applies the DADA2 pipeline to raw paired-end Illumina (NextSeq) amplicon reads to learn a sample-agnostic error model, denoise reads, merge pairs, and remove chimeras, yielding per-marker MOTU count tables for metabarcoding markers such as mitochondrial 16S rRNA and COI.

## When to use

Use when raw paired-end Illumina FASTQ amplicon reads (e.g. mitochondrial 16S rRNA or COI metabarcoding) must be denoised into per-marker MOTU count tables with error modelling and chimera removal.

## When NOT to use

- Input is already a denoised or feature/MOTU count table rather than raw paired-end FASTQ reads
- Reads are single-end or the amplicon design cannot produce at least a 5 bp overlap between forward and reverse reads
- A reference-based (closed-labeling) OTU clustering approach is required instead of de novo sequence-inference denoising

## Inputs

- Raw paired-end Illumina NextSeq FASTQ files per sample, separated by marker (16S rRNA and COI amplicons)

## Outputs

- Per-marker MOTU count tables (e.g. motu_count_table.tsv) with 1,162 MOTUs for COI and 843 MOTUs for 16S
- Chimera-filtered merged sequence table

## How to apply

Load the deposited raw NextSeq paired-end FASTQ files per sample with the dada2 R package and apply dada2 filterAndTrim to each file. Learn a sample-agnostic error model with learnErrors and apply dada to denoise reads separately for the 16S and COI amplicons, keeping the two markers distinct throughout. Merge forward and reverse denoised reads with mergePairs requiring a minimum 5 bp overlap between read pairs, then remove chimeric sequences with removeBimeraDenovo on the merged sequence table. Summarize the chimera-filtered sequence table into MOTU count tables per marker and write the combined table to motu_count_table.tsv. Validate by comparing per-marker MOTU totals against published values (e.g. 1,162 COI MOTUs and 843 16S MOTUs) within reasonable tolerance.

## Related tools

- **DADA2** (R package used for filterAndTrim, learnErrors, dada denoising, mergePairs (min 5 bp overlap) and removeBimeraDenovo to infer MOTUs from raw NextSeq reads) — tobiasgf/Bioinformatic-tools
- **R** (Scripting environment in which the DADA2 workflow is run to produce the MOTU count table)

## Examples

```
Rscript -e 'lib(dada2); dF<-dada(fqF,errF); dR<-dada(fqR,errR); m<-mergePairs(dF,dR,minOverlap=5); nt<-removeBimeraDenovo(m); write.table(nt,"motu_count_table.tsv",sep="\t")'
```

## Evaluation signals

- Per-marker MOTU totals match the published values within reasonable tolerance: 1,162 MOTUs for COI and 843 MOTUs for 16S
- All merged read pairs satisfy the minimum 5 bp forward-reverse overlap requirement
- Chimeric sequences have been removed via removeBimeraDenovo before summarizing counts
- 16S and COI amplicons were denoised with separate, marker-specific error models rather than a shared model
- Output file motu_count_table.tsv exists and contains a per-marker, per-sample MOTU count matrix

## Limitations

- The mergePairs step requires at least 5 bp overlap between read pairs; shorter overlaps cause read loss
- MOTU counts are validated against this specific study's numbers (1,162 COI / 843 16S); other datasets will yield different totals and should be judged by their own expectations
- Error models are sample-agnostic and must be learned separately for each marker amplicon

## Evidence

- [abstract] Primer sets targeting mitochondrial 16S rRNA and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] DADA2 workflow with 5 bp merge overlap and chimera removal: [evidence span withheld — anchor status: not_found]
- [other] Expected per-marker MOTU totals: [evidence span withheld — anchor status: not_found]
