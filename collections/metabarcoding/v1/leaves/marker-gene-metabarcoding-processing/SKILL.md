---
name: marker-gene-metabarcoding-processing
description: Use when starting from raw paired-end NextSeq FASTQ files of 16S or COI
  amplicons and the goal is an error-filtered, merged, chimera-removed MOTU count
  table per marker gene.
when_to_use_negative:
- Input is already a demultiplexed, denoised sequence table or MOTU/ASV abundance
  matrix rather than raw FASTQ reads
- The amplicon targets nuclear loci or non-mitochondrial markers for which the 16S/COI-specific
  error models and 5 bp merge-overlap assumptions of this workflow were built
edam_operation: http://edamontology.org/operation_3237
edam_topics: []
tools:
- name: DADA2
  role: R package performing filterAndTrim, learnErrors, dada denoising, mergePairs
    (min 5 bp overlap), and removeBimeraDenovo chimera removal to build MOTU tables
    from raw 16S/COI reads
  repo: https://github.com/tobiasgf/Bioinformatic-tools
- name: R
  role: Runtime environment in which the DADA2 pipeline steps are executed end-to-end
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:57ea929cbcb84b01091725f099e821fa848d51a57cd3efbdf59e4cb7bab0b976
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - R
  edam_operation: http://edamontology.org/operation_3237
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# marker-gene-metabarcoding-processing

## Summary

Denoises raw paired-end Illumina amplicon reads (mitochondrial 16S rRNA and COI) into error-corrected MOTU count tables using the DADA2 R package, as applied to flower eDNA metabarcoding of terrestrial arthropods.

## When to use

Use when starting from raw paired-end NextSeq FASTQ files of 16S or COI amplicons and the goal is an error-filtered, merged, chimera-removed MOTU count table per marker gene.

## When NOT to use

- Input is already a demultiplexed, denoised sequence table or MOTU/ASV abundance matrix rather than raw FASTQ reads
- The amplicon targets nuclear loci or non-mitochondrial markers for which the 16S/COI-specific error models and 5 bp merge-overlap assumptions of this workflow were built

## Inputs

- Raw paired-end Illumina NextSeq FASTQ files per sample, split by amplicon (16S rRNA and COI)
- Primer/marker assignment distinguishing the 16S and COI libraries

## Outputs

- Per-marker MOTU count table (chimera-filtered merged sequence table)
- Combined MOTU count table written to motu_count_table.tsv

## How to apply

Load the deposited raw paired-end FASTQ files per sample with the dada2 R package and apply filterAndTrim to each file. Learn a sample-agnostic error model with learnErrors and denoise forward and reverse reads separately with dada for the 16S and COI amplicons. Merge denoised pairs with mergePairs requiring a minimum 5 bp overlap, then remove chimeras with removeBimeraDenovo on the merged sequence table. Summarize the chimera-filtered sequence table into per-marker MOTU count tables (expected: 1,162 MOTUs for COI and 843 for 16S in the reference run) and write the combined table to motu_count_table.tsv.

## Related tools

- **DADA2** (R package performing filterAndTrim, learnErrors, dada denoising, mergePairs (min 5 bp overlap), and removeBimeraDenovo chimera removal to build MOTU tables from raw 16S/COI reads) — https://github.com/tobiasgf/Bioinformatic-tools
- **R** (Runtime environment in which the DADA2 pipeline steps are executed end-to-end)

## Examples

```
library(dada2); mergers <- mergePairs(dada(derepF, errF), derepF, dada(derepR, errR), derepR, minOverlap=5); seqtab <- removeBimeraDenovo(mergers, method="consensus")
```

## Evaluation signals

- Per-marker MOTU totals match the published values: 1,162 MOTUs for COI and 843 MOTUs for 16S, within reasonable tolerance
- Every merged read pair satisfies the minimum 5 bp forward-reverse overlap requirement before counting
- The final chimera-filtered sequence table contains no bimeras flagged by removeBimeraDenovo
- The combined output file motu_count_table.tsv exists and contains MOTU counts separated per marker gene (COI vs 16S)
- Denoising was run separately for the 16S and COI amplicons, each with its own learned error model

## Limitations

- The reference run's expected MOTU counts (1,162 COI, 843 16S) are dataset-specific; other datasets will not reproduce them exactly
- Requiring a minimum 5 bp overlap in mergePairs can drop read pairs whose forward and reverse reads fail to align
- The underlying eDNA experiment was not exhaustive — the article notes that 'an even higher arthropod richness could be obtained using this eDNA approach'

## Evidence

- [abstract] two generic primers targeting mitochondrial markers: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] DADA2 pipeline MOTU totals per marker: [evidence span withheld — anchor status: not_found]
- [other] minimum 5 bp overlap in mergePairs: [evidence span withheld — anchor status: not_found]
- [abstract] richness not exhaustive: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
