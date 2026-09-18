---
name: paired-read-merging-minimum-overlap
description: Use when paired-end Illumina NextSeq reads from COI or 16S amplicons
  have already been denoised with dada2 and you need to merge forward/reverse reads
  into full-length sequences while controlling overlap stringency.
when_to_use_negative:
- Reads have not yet been denoised with dada2's dada step — mergePairs expects dada-class
  inputs, not raw FASTQ
- Single-end sequencing data or amplicons whose forward and reverse reads do not genuinely
  overlap — a minimum-overlap merge is meaningless without paired-end overlap
- Input is already a merged or chimera-filtered MOTU/feature table — re-merging would
  double-process the data
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: 'mergePairs step: merges denoised forward and reverse COI/16S reads with a
    minimum 5 bp overlap to build the merged sequence table'
  repo: https://github.com/tobiasgf/Bioinformatic-tools
- name: R
  role: Execution environment for the dada2 package workflow (filterAndTrim, learnErrors,
    dada, mergePairs)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:dea1e7cedb5609df606ff6c5238ad63434fa261973aacc83a1690ec6251b18ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - R
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# paired-read-merging-minimum-overlap

## Summary

Merges DADA2-denoised forward and reverse amplicon reads into full-length MOTU sequences by requiring a minimum 5 bp overlap between read pairs, as used in the flower eDNA metabarcoding of COI and 16S mitochondrial markers.

## When to use

Use when paired-end Illumina NextSeq reads from COI or 16S amplicons have already been denoised with dada2 and you need to merge forward/reverse reads into full-length sequences while controlling overlap stringency.

## When NOT to use

- Reads have not yet been denoised with dada2's dada step — mergePairs expects dada-class inputs, not raw FASTQ
- Single-end sequencing data or amplicons whose forward and reverse reads do not genuinely overlap — a minimum-overlap merge is meaningless without paired-end overlap
- Input is already a merged or chimera-filtered MOTU/feature table — re-merging would double-process the data

## Inputs

- DADA2-denoised forward read objects (dada-class) per sample from COI or 16S amplicons
- DADA2-denoised reverse read objects (dada-class) per sample
- Dereplicated per-sample read objects (derep-class) from raw NextSeq paired-end FASTQ files

## Outputs

- Per-sample merged read-pair data frame (merged-class) with sequence, abundance, forward/reverse indices, nmatch, nmismatch, nindel, prefer, nmatchPWM, and acceptance flags
- Merged sequence table suitable for chimera filtering with removeBimeraDenovo

## How to apply

After running dada2's filterAndTrim on each paired-end FASTQ file and learning the sample-agnostic error model with learnErrors, denoise the 16S and COI amplicons separately with the dada function. Apply mergePairs to the denoised forward and reverse reads, requiring a minimum 5 bp overlap between read pairs — this threshold is strict enough to reject spurious alignments between non-overlapping COI/16S read pairs while retaining true amplicons. Pass the per-sample dereplicated read objects so that abundance counts propagate to the merged sequences. Downstream, feed the merged sequence table to removeBimeraDenovo for chimera removal before building the final MOTU count table (expecting 1,162 COI MOTUs and 843 16S MOTUs in the reference run).

## Related tools

- **DADA2** (mergePairs step: merges denoised forward and reverse COI/16S reads with a minimum 5 bp overlap to build the merged sequence table) — https://github.com/tobiasgf/Bioinformatic-tools
- **R** (Execution environment for the dada2 package workflow (filterAndTrim, learnErrors, dada, mergePairs))

## Examples

```
merged <- mergePairs(ddF, ddR, derepF, derepR, minOverlap = 5, maxMismatch = 0)
```

## Evaluation signals

- Merged read counts should be consistent with (i.e., not exceed) the denoised forward/reverse read counts per sample; merging discards non-overlapping pairs
- Chimera-filtered MOTU totals after merging should match the published reference values (1,162 COI MOTUs, 843 16S MOTUs) within reasonable tolerance
- All accepted merged sequences should show nmatch/nmismatch and overlap lengths consistent with the minOverlap=5 setting and no large indels (nindel near zero)
- Merged amplicon lengths should fall within the expected COI and 16S primer-region size ranges rather than concatenating non-overlapping read pairs
- The final motu_count_table.tsv should contain per-marker count tables only for samples with successfully merged reads

## Limitations

- The 5 bp minimum overlap threshold is a strict cutoff; pairs with genuine overlap shorter than 5 bp are silently discarded, potentially reducing MOTU recovery
- The run was validated only against the published MOTU totals for this specific COI/16S flower-eDNA dataset; other markers or read lengths may need a different overlap threshold
- Merging alone does not remove chimeras — a subsequent removeBimeraDenovo step is still required before counting MOTUs

## Evidence

- [other] Merge forward and reverse denoised reads with mergePairs, requiring a minimum 5 bp overlap between read pairs.: [evidence span withheld — anchor status: not_found]
- [abstract] eDNA metabarcoding of flowers using 16S and COI primers: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] Denoise reads separately for the 16S and COI amplicons: [evidence span withheld — anchor status: not_found]
