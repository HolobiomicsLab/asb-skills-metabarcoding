---
name: motu-count-table-generation
description: Use when starting from deposited raw paired-end NextSeq FASTQ files of
  COI and 16S amplicons and the goal is a denoised, chimera-filtered MOTU abundance
  table with per-marker MOTU totals.
when_to_use_negative:
- Input is already a denoised feature/MOTU table or OTU count matrix — no FASTQ denoising
  is needed
- Amplicons are non-mitochondrial markers (e.g. ITS, 18S) with very different read
  lengths and error profiles than the COI/16S setup described here
- Only single-end reads are available — the mergePairs step requiring a 5 bp forward/reverse
  overlap cannot be applied
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: DADA2
  role: R package used for filterAndTrim, learnErrors/dada read denoising, mergePairs
    (min 5 bp overlap), and removeBimeraDenovo chimera removal to build per-marker
    MOTU count tables
  repo: tobiasgf/Bioinformatic-tools
- name: R
  role: Scripting environment in which the dada2 pipeline is executed and the combined
    motu_count_table.tsv is written
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:da1466675b157177d7c9291f14fade49cb26dc9ed54617a1604e6429ce0974fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - R
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# motu-count-table-generation

## Summary

Generates marker-specific MOTU (metabarcoding OTU) count tables from raw paired-end Illumina amplicon reads by running the DADA2 pipeline (filterAndTrim, learnErrors/dada, mergePairs, removeBimeraDenovo) separately per mitochondrial marker, producing a motu_count_table.tsv with per-sample MOTU abundances for COI and 16S amplicons.

## When to use

Use when starting from deposited raw paired-end NextSeq FASTQ files of COI and 16S amplicons and the goal is a denoised, chimera-filtered MOTU abundance table with per-marker MOTU totals.

## When NOT to use

- Input is already a denoised feature/MOTU table or OTU count matrix — no FASTQ denoising is needed
- Amplicons are non-mitochondrial markers (e.g. ITS, 18S) with very different read lengths and error profiles than the COI/16S setup described here
- Only single-end reads are available — the mergePairs step requiring a 5 bp forward/reverse overlap cannot be applied

## Inputs

- Raw paired-end Illumina NextSeq FASTQ files per sample, split by amplicon (mitochondrial COI and 16S rRNA)
- Marker assignment (which FASTQs belong to the 16S vs COI amplicon)

## Outputs

- motu_count_table.tsv — combined MOTU-by-sample abundance table with per-marker MOTU totals (1,162 COI MOTUs, 843 16S MOTUs)
- Per-marker chimera-filtered dada2 sequence tables

## How to apply

Load the deposited raw NextSeq paired-end FASTQ files per sample with the dada2 R package and apply filterAndTrim to each file. Learn a sample-agnostic error model with learnErrors and run dada to denoise reads separately for the 16S and COI amplicons, keeping the two markers in distinct runs so their different read lengths and error profiles do not contaminate each other's model. Merge forward and reverse denoised reads with mergePairs, requiring a minimum 5 bp overlap between read pairs. Remove chimeric sequences with removeBimeraDenovo on the merged sequence table, then summarize the chimera-filtered table into MOTU count tables per marker and write the combined table to motu_count_table.tsv. For the reference dataset this yields 1,162 MOTUs for COI and 843 MOTUs for 16S; accept the run if per-marker totals match these published numbers within reasonable tolerance.

## Related tools

- **DADA2** (R package used for filterAndTrim, learnErrors/dada read denoising, mergePairs (min 5 bp overlap), and removeBimeraDenovo chimera removal to build per-marker MOTU count tables) — tobiasgf/Bioinformatic-tools
- **R** (Scripting environment in which the dada2 pipeline is executed and the combined motu_count_table.tsv is written)

## Examples

```
Rscript -e 'library(dada2); m <- mergePairs(dada(filts, err=errCOI), minOverlap=5); st <- removeBimeraDenovo(m, method="consensus"); write.table(st, "motu_count_table.tsv", sep="\t")'
```

## Evaluation signals

- Per-marker MOTU totals match the published values: 1,162 MOTUs for COI and 843 MOTUs for 16S, within reasonable tolerance
- mergePairs was run with a minimum 5 bp overlap between forward and reverse read pairs
- removeBimeraDenovo was applied to the merged sequence table, and the chimera-filtered table (not the pre-chimera table) is what is summarized
- Error models were learned and applied separately for the 16S and COI amplicons, not pooled across markers
- Output motu_count_table.tsv parses as a MOTU-by-sample integer count table with both markers represented

## Limitations

- The pipeline expects paired-end reads with sufficient forward/reverse overlap (at least 5 bp); markers without overlap cannot be merged
- The 1,162 COI / 843 16S MOTU counts are specific to this dataset's sequencing depth and primer sets; other datasets will produce different totals, so the validation is against per-marker consistency and chimera filtering rather than the exact numbers
- The experiment was not exhaustive — higher arthropod richness could be obtained with this eDNA approach — so the MOTU table underestimates true community richness

## Evidence

- [abstract] eDNA metabarcoding of flowers using two primer sets targeting mitochondrial 16S rRNA and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] eDNA recovered from at least 135 arthropod species in 67 families and 14 orders: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] Non-exhaustive sampling implies higher arthropod richness is obtainable: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
