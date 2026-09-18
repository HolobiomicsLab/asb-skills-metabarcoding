---
name: chimera-removal-amplicon-sequencing
description: Use when paired-end Illumina amplicon FASTQ data (e.g. mitochondrial
  16S rRNA and COI) has been denoised and merged with dada2 and chimeric sequences
  must be removed before building MOTU count tables.
when_to_use_negative:
- Input is already a chimera-filtered MOTU/ASV count table rather than merged dada2
  sequence tables
- Data are not paired-end amplicon reads processed through dada2 filterAndTrim, learnErrors/dada,
  and mergePairs (e.g. single-end reads with no mergePairs step)
- The goal is metabarcoding primer design or taxonomic assignment rather than sequence-table
  curation
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: R package providing filterAndTrim, learnErrors, dada, mergePairs (min 5 bp
    overlap) and removeBimeraDenovo for denoising, merging, and chimera removal of
    the 16S and COI amplicon reads
- name: R
  role: Environment in which the dada2 pipeline scripts are run to produce motu_count_table.tsv
  repo: https://github.com/tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:f6179e042bb1029a93a9ad8da00b0d2f8e3c3657f5a11e65b9de3206b7c44d42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - R
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# chimera-removal-amplicon-sequencing

## Summary

Removes chimeric amplicon sequences from a merged, denoised sequence table using dada2's removeBimeraDenovo, producing per-marker MOTU count tables (here 1,162 COI and 843 16S MOTUs from raw Illumina NextSeq reads).

## When to use

Use when paired-end Illumina amplicon FASTQ data (e.g. mitochondrial 16S rRNA and COI) has been denoised and merged with dada2 and chimeric sequences must be removed before building MOTU count tables.

## When NOT to use

- Input is already a chimera-filtered MOTU/ASV count table rather than merged dada2 sequence tables
- Data are not paired-end amplicon reads processed through dada2 filterAndTrim, learnErrors/dada, and mergePairs (e.g. single-end reads with no mergePairs step)
- The goal is metabarcoding primer design or taxonomic assignment rather than sequence-table curation

## Inputs

- Raw paired-end Illumina NextSeq FASTQ files per sample (16S rRNA and COI amplicons)
- Denoised and merged dada2 sequence table (output of mergePairs with min overlap 5 bp)

## Outputs

- Chimera-filtered MOTU count table per marker (COI: 1,162 MOTUs; 16S: 843 MOTUs)
- Combined MOTU count table written to motu_count_table.tsv

## How to apply

Load the deposited raw NextSeq paired-end FASTQ files per sample into R with the dada2 package and apply filterAndTrim, then learn a sample-agnostic error model with learnErrors and denoise with dada separately for the 16S and COI amplicons. Merge forward and reverse denoised reads with mergePairs, requiring a minimum 5 bp overlap between read pairs. Run dada2 removeBimeraDenovo on the merged sequence table to flag and drop chimeric sequences. Summarize the chimera-filtered sequence table into per-marker MOTU count tables and write the combined table to motu_count_table.tsv; validate by comparing per-marker MOTU totals against the reported 1,162 COI and 843 16S MOTUs within reasonable tolerance.

## Related tools

- **DADA2** (R package providing filterAndTrim, learnErrors, dada, mergePairs (min 5 bp overlap) and removeBimeraDenovo for denoising, merging, and chimera removal of the 16S and COI amplicon reads)
- **R** (Environment in which the dada2 pipeline scripts are run to produce motu_count_table.tsv) — https://github.com/tobiasgf/Bioinformatic-tools

## Examples

```
chimera_free <- dada2::removeBimeraDenovo(mergedSeqTab, method="consensus", multithread=TRUE); write.table(chimera_free, "motu_count_table.tsv", sep="\t")
```

## Evaluation signals

- Per-marker MOTU totals from the chimera-filtered table match the published values: 1,162 MOTUs for COI and 843 MOTUs for 16S within reasonable tolerance
- Chimera removal is applied only after mergePairs succeeded with the required minimum 5 bp overlap between read pairs
- Output file motu_count_table.tsv exists and contains the combined MOTU count table for both markers
- Denuded (chimeric) sequences are removed from the merged sequence table before summarization, so no bimeras remain in the final MOTU table
- Per-sample read counts decrease monotonically through filtering, denoising, merging, and chimera removal, with no sample gaining reads at any step

## Limitations

- The expected MOTU counts (1,162 COI, 843 16S) serve as validation targets, so runs deviating substantially from these totals indicate the pipeline (filtering, merging, or chimera removal) was misapplied
- The 5 bp minimum overlap requirement in mergePairs can discard read pairs with short amplicon overlaps, reducing merged reads before chimera removal
- The error model is sample-agnostic (learnErrors applied across samples), which may not capture marker-specific error profiles if COI and 16S are not denoised separately

## Evidence

- [abstract] primer sets targeting mitochondrial markers: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] mergePairs minimum overlap: [evidence span withheld — anchor status: not_found]
- [other] removeBimeraDenovo chimera removal: [evidence span withheld — anchor status: not_found]
