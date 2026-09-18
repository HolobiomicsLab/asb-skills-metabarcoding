---
name: taxonomic-assignment-genus-collapsing
description: Use when denoised 16S rRNA ASVs (e.g., Deblur output from paired-end
  Illumina reads of sputum samples) need genus-level taxonomy assignments comparable
  across hypervariable regions such as V1–V2, V3–V4, V5–V7, and V7–V9.
when_to_use_negative:
- Input sequences come from full-length 16S rRNA or third-generation (e.g., Nanopore)
  sequencing, where the Illumina MiSeq 300 bp constraint and Greengenes genus-level
  training do not apply
- The analysis requires species- or strain-level resolution, which the genus-level
  Greengenes collapse intentionally discards
- Input is already a taxonomy-collapsed feature table or the target classification
  standard is SILVA rather than Greengenes
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3168
tools:
- name: Deblur
  role: Denoises paired-end 16S rRNA reads and identifies bacterial amplicon sequence
    variants (ASVs) that are subsequently classified at the genus level
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database from which taxonomic assignments are obtained and collapsed
    at the genus level
- name: QIIME 2
  role: Supplies the non-chimeric 16S rRNA database used to identify chimeras in the
    reads before taxonomy assignment (best hit similarity < 99%)
- name: FastQC
  role: Quality assessment and filtering of raw reads (Q30 criterion) prior to denoising
    and classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:a468bcebdffbcad9b57a2fb9e830826f2a4ac855f4679340b5734d22dd39953d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - Illumina MiSeq
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# taxonomic-assignment-genus-collapsing

## Summary

Assign taxonomy to denoised 16S rRNA amplicon sequence variants (ASVs) from respiratory sputum samples using the Greengenes database collapsed at the genus level, after Deblur denoising and QIIME 2-based chimera removal, to enable cross-region comparison of bacterial genera across hypervariable regions.

## When to use

Use when denoised 16S rRNA ASVs (e.g., Deblur output from paired-end Illumina reads of sputum samples) need genus-level taxonomy assignments comparable across hypervariable regions such as V1–V2, V3–V4, V5–V7, and V7–V9.

## When NOT to use

- Input sequences come from full-length 16S rRNA or third-generation (e.g., Nanopore) sequencing, where the Illumina MiSeq 300 bp constraint and Greengenes genus-level training do not apply
- The analysis requires species- or strain-level resolution, which the genus-level Greengenes collapse intentionally discards
- Input is already a taxonomy-collapsed feature table or the target classification standard is SILVA rather than Greengenes

## Inputs

- Denoised 16S rRNA ASV sequences (Deblur output from paired-end Illumina MiSeq reads, ~300 bp limit)
- Raw FASTQ read files from 33 sputum samples (per repository github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification)
- Greengenes 16S rRNA reference database for taxonomic classification
- QIIME 2 non-chimeric 16S rRNA database for chimera checking

## Outputs

- Genus-level taxonomy assignments for each ASV per hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)
- Count of distinct taxonomies per region (e.g., 114 taxonomies reported across regions)
- Taxonomy-collapsed feature tables supporting relative abundance comparisons of bacterial genera

## How to apply

After quality filtering raw reads with FastQC (Version 0.11.9) and denoising with the Deblur pipeline (intended for paired-end data, with non-redundant reads deleted), identify chimeras using the non-chimeric 16S rRNA database from QIIME 2, flagging reads with a best hit similarity rate of less than 99%. Then obtain taxonomic assignments using the Greengenes database and collapse them at the genus level, so relative abundances of bacterial genera can be compared pairwise across the combined hypervariable regions (V1–V2, V3–V4, V5–V7, V7–V9). The genus-level collapse is the decision point that standardizes resolution for downstream cross-validation against the ZymoBIOMICS mock community control and ROC-based accuracy assessment. Note the article's caveat that regions were trained with Greengenes despite more updated SILVA versions being available.

## Related tools

- **Deblur** (Denoises paired-end 16S rRNA reads and identifies bacterial amplicon sequence variants (ASVs) that are subsequently classified at the genus level) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database from which taxonomic assignments are obtained and collapsed at the genus level)
- **QIIME 2** (Supplies the non-chimeric 16S rRNA database used to identify chimeras in the reads before taxonomy assignment (best hit similarity < 99%))
- **FastQC** (Quality assessment and filtering of raw reads (Q30 criterion) prior to denoising and classification)

## Evaluation signals

- All ASVs across the four region combinations (V1–V2, V3–V4, V5–V7, V7–V9) resolve to genus-level labels after Greengenes collapsing; no higher-rank-only or unassigned majority table is produced
- Chimeric reads flagged against the QIIME 2 non-chimeric database with best hit similarity below 99% are excluded before counting taxonomies
- Reproduced taxonomy count matches the reported figure of 114 taxonomies and per-region ASV totals (~1,568,227 ASVs per hypervariable region from ~100,000 sequences per sample over 33 sputum samples)
- Cross-validation against the ZymoBIOMICS mock community (10 species) yields genus-level concordance consistent with reported sensitivity/specificity rankings (highest for V1–V2)

## Limitations

- Regions were trained with Greengenes despite the availability of more updated SILVA versions, which may bias taxonomic assignment
- Genus-level collapsing limits resolution; the mock microbial community contained only 10 species, which affects the sensitivity and specificity of taxonomic classification
- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling and therefore the taxonomic completeness achievable per region

## Evidence

- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [methods] Chimera filtering threshold: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [methods] Deblur denoising before assignment: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [abstract] ASV identification at genus level: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [discussion] Reference database caveat: "despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes"
